# AST Composition

To build our generator, we start out by identifying the key components of our DFM files within the subset. Based on them, we simultaneously implement the *AST* generator's components.

## Component Node

The easiest first step is to determine the shape of our components. Thus, we create a `ComponentNode.java` class, and look at our grammar:

```yaml
grammar DelphiDFM ;

file             : block+ EOF 
                 ;

// Parser Rules
block            : ('inherited' | 'object' | 'inline') ID ':' ID propertyBlock* block* 'end'   # beginBlock
                 ;
```

We know that our DFMs are composed of `block`s, which themselves are composed of keywords, strings, potential children as `propertyBlocks` (which hold the same structure), or other nested `block`s. Thus, we define three public strings: `keyword`, `name`, and `type`. We complement these with the addition of `role`, which is of type `ComponentRoleEnum` (more on that in the next page: [AST Node Classification](ast-gen/node-spec.md)).

Next, to account for nested blocks and property blocks, we instantiate lists of both:

```java
public List<PropertyNode> properties = new ArrayList<>();
public List<ComponentNode> children = new ArrayList<>();
```

**Tip:** if you ever feel lost, always refer to the grammar and the generated parser. These two should hold all the necessary steps which need to be taken.

## Property Node

Continuing to follow our grammar, we follow up with our `PropertyNode.java` implementation. Let's look at our grammar again:

```yaml
propertyBlock    : SUBPROPERTY '=' value        # subProperty
                 | ID '=' value                 # valueProperty
                 ;
```

This one is also fairly simple. Our properties can be one of two choices: either an `ID` token (meaning String), or a `SUBPROPERTY` token (meaning Strings concatenated with dots '.').

Therefore, we create a name for properties, and a value object. Just like we did with `ComponentNode`, we supplement property nodes with classifications as well, through the inclusion of `PropertyCategoryEnum` (similarily we discuss these on the following page, [AST Node Classification](ast-gen/node-spec.md)).

```java
public String name;    
public Object value;
public PropertyCategoryEnum category;

public boolean isNamespaced() {
    return name.contains(".");
}
```

We also make a distinction between simple and namespaced strings. However, note that the `isNamespaced()` boolean does not get used at all when generating *ASTs*. This will become useful later on during the implementation of the transformation layer, by simply checking its returned value, rather than re-implementing the dot.

There is one more thing we add here, which is a `toString()` function:

```java
@Override
public String toString() {
    return "[" + category + "] " + name + " = " + value;
}
```

This function will become relevant when we need to print properties within collection items.

## Item List Node

This is where things get a tad more complicated, and also where we (indirectly) make use of the `toString()` function of `PropertyNode`.

Within DFM files, collection item lists take the following form:

```yaml
object collItemExample : TcxGridDBColumn
    Properties.ListColumns = <
      item
        FieldName = 'fullName'
      end>
end
```

Let's see the structure we defined in our grammar:

```yaml
collectionItems  : '<' item* '>' ;
item             : 'item' propertyBlock* 'end' ;
```

To account for this structure, we instantiate a matrix of type `PropertyNode`:

```java
public class ItemListNode {
    public List<List<PropertyNode>> items = new ArrayList<>();

}
```

Then, we stringify the contents within collection items and enclose them however we like (in this case "ItemList{}"). We also append an identifier element to each item, and indicate its index (in this case "item[index]"). Within, we loop through properties and append each, with additional indents to indicate hierarchy.

```java
public class ItemListNode {
    public List<List<PropertyNode>> items = new ArrayList<>();

    @Override
    public String toString() {
        StringBuilder strBuild = new StringBuilder("ItemList{\n");
        for (int i = 0; i < items.size(); i++) {
            strBuild.append("  item[").append(i).append("]:\n");
            for (PropertyNode prop : items.get(i)) {
                strBuild.append("    ").append(prop).append("\n");
            }
        }
        strBuild.append("}");
        return strBuild.toString();
    }
}
```

When we call `.append(prop)` on a `StringBuilder`, java implicitly calls `prop.toString()`, thus we get the property structure returned by the function.

## DFM Value

The last composite of our *AST* will be the specification of values. Let's examine our grammar once again to determine what needs to be handled:

```yaml
value       : INT                    # intValue
            | FLOAT                  # floatValue
            | STRING                 # stringLiteralValue
            | ID                     # stringValue
            | ARRAY                  # arrayValue
            | collectionItems        # collectionItemValue
            | parenList              # parenListValue
            | SUBPROPERTY            # dottedIdValue
            ;

...

parenList   : '(' (INT | FLOAT | STRING)* ')' ;
```

We handle these options easily, by specifying all alternatives as enums in our `DfmValue` class. With the value types defined, we create a typed wrapper that pairs each value with its corresponding type. This allows us to preserve type information throughout the *AST*, which becomes important when the transformation layer needs to emit values differently based on their type. For instance, strings need quotes in the target language while integers do not.

```java
public class DfmValue {
    public enum Type {
        STRING, INTEGER, FLOAT, IDENTIFIER, ARRAY, ITEM_LIST, PAREN_LIST, DOTTED_ID
    }

    public final Type type;
    public final Object raw;

    public DfmValue(Type type, Object raw) {
        this.type = type;
        this.raw = raw;
    }

}
```

The raw field is typed as Object because the actual Java type varies per enum value: String for `STRING` and `IDENTIFIER`, Integer for `INTEGER`, Double for `FLOAT`, and `List<DfmValue>` for `PAREN_LIST`. The type enum tells consumers how to interpret raw without casting blindly.

We also provide a `toString()` for readable *AST* output. Strings are re-wrapped in single quotes to mirror the original DFM notation, and parenthesized lists are enclosed in parentheses:

```java
public class DfmValue {
    ...

    @Override
    public String toString() {
        return switch (type) {
            case STRING -> "'" + raw + "'";
            case PAREN_LIST -> "(" + raw + ")";
            case DOTTED_ID -> String.valueOf(raw);
            default -> String.valueOf(raw);
        };
    }
}
```

**Note** the `PAREN_LIST` type. This handles DFM constructs where multiple scalar values are grouped inside parentheses, separated by newlines. For instance:

```yaml
DesignSize = (
    220
    110)
```

In our AST, the raw field for a `PAREN_LIST` value contains a `List<DfmValue>`, where each element is itself a typed value (in this case, two `INTEGERs`). This recursive use of `DfmValue` within a list allows us to preserve both the grouping structure and the individual types of each element.

### Conclusion

Now we have covered the full structure of our tokens, with the exception of property and component classifications. On the next page we detail how we tackled these classifications, which represent the final step of refining our *AST* node notations.