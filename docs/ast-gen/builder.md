# Implementing ASTBuilder

It's time to levergae all of our components and hook them up to a custom java class. This is where we make use of the *ANTLR-generated* methods, which were based off of our grammar annotations from the [Grammar Update](ast-gen/grammar-update.md) page.

## Visit File

We create an `ASTBuilder` java class which extends the generated `DelphiDFMBaseVisitor`:

```java
import java.util.ArrayList;
import java.util.List;

import org.antlr.v4.runtime.tree.TerminalNode;

public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    @Override
    public Object visitFile(DelphiDFMParser.FileContext ctx) {
        List<ComponentNode> roots = new ArrayList<>();
        for (var block : ctx.block()) {
            roots.add((ComponentNode) visit(block));
        }
        return roots;
    }

    ...

}
```

We visit each `block` token, and append them to a list of type `ComponentNode`.

## Visit Blocks

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitBeginBlock(DelphiDFMParser.BeginBlockContext ctx) {
        ComponentNode node = new ComponentNode();

        node.keyword = ctx.getChild(0).getText(); 
        node.name = ctx.ID(0).getText(); 
        node.typeName = ctx.ID(1).getText();
        node.role = ComponentClassifier.classify(node.typeName);

        for (var propB : ctx.propertyBlock()) {
            Object result = visit(propB);
            if (result instanceof PropertyNode prop) {
                node.properties.add(prop);
            }
        }

        for (var child : ctx.block()) {
            node.children.add((ComponentNode) visit(child));
        }

        return node;
    }

    ...

}
```

We start by instantiating a `ComponentNode`, and assign the consumed tokens to each object.

Then, we visit each property block, check if it's of type `PropertyNode`, and append to the end of the list. We also loop through nested children, and append the visited children to `ComponentNode`. Then finally, we return everything.

## Visit Sub-property

We tackle sub-properties by instantiating a `PropertyNode` object, and assigning the consumed tokens based on what is expected.

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitSubProperty(DelphiDFMParser.SubPropertyContext ctx) {
        PropertyNode node = new PropertyNode();
        node.name = ctx.SUBPROPERTY().getText();
        node.value = visit(ctx.value());
        node.category = PropertyClassifier.classify(node.name);
        return node;
    }

    ...

}
```

## Visit String Value

This approach is similar to the previous one, with the exception that it focuses on strings.

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitValueProperty(DelphiDFMParser.ValuePropertyContext ctx) {
        PropertyNode node = new PropertyNode();
        node.name = ctx.ID().getText();
        node.value = visit(ctx.value());
        node.category = PropertyClassifier.classify(node.name);
        return node;
    }

    ...

}
```

## Visit Value Alternatives

Now we go through our `DfmValue` types and get their contents:

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitStringLiteralValue(DelphiDFMParser.StringLiteralValueContext ctx) {
        String raw = ctx.STRING().getText();
        // single quotes re-printed in DfmValue
        return new DfmValue(DfmValue.Type.STRING, raw.substring(1, raw.length() - 1));
    }

    @Override
    public Object visitIntValue(DelphiDFMParser.IntValueContext ctx) {
        return new DfmValue(DfmValue.Type.INTEGER, Integer.parseInt(ctx.INT().getText()));
    }

    ...

}
```

## Visit Collection Items

First, we get the collection item structure. Tracing back to the parser, the `CollectionItemValueContext` returns our `collectionItem` rule:

```yaml
collectionItems  : '<' item* '>' ;
item             : 'item' propertyBlock* 'end' ;
```

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitCollectionItemValue(DelphiDFMParser.CollectionItemValueContext ctx) {
        return visit(ctx.collectionItems());
    }

    ...

}
```

Then, we visit the item declarations themselves, and append them to a `PropertyNode` matrix from our `ItemListNode`:

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitCollectionItems(DelphiDFMParser.CollectionItemsContext ctx) {
        ItemListNode node = new ItemListNode();
        for (var item : ctx.item()) {
            node.items.add((List<PropertyNode>) visit(item));
        }
        return node;
    }

    ...

}
```

Finally, we go through each item property and append them as `PropertyNode` to a list:

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitItem(DelphiDFMParser.ItemContext ctx) {
        List<PropertyNode> props = new ArrayList<>();
        for (var propB : ctx.propertyBlock()) {
            Object result = visit(propB);
            if (result instanceof PropertyNode p) {
                props.add(p);
            }
        }
        return props;
    }

    ...

}
```

## Visit Parentheses List

The last step is to tackle our `parenList` rule.

```yaml
parenList : '(' (INT | FLOAT | STRING)* ')' ;
```

We start by visiting the `parenList` context:

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitParenListValue(DelphiDFMParser.ParenListValueContext ctx) {
        return visit(ctx.parenList());
    }

    ...

}
```

Then, we collect the scalar values inside parentheses. We use `ctx.children` to retain the order of values, otherwise they would be mixed during token output. Afterwards, we implement a couple of checks, to determine the value type, based on the token type, using `DfmValue`.

```java
public class ASTBuilder extends DelphiDFMBaseVisitor<Object> {

    ...

    @Override
    public Object visitParenList(DelphiDFMParser.ParenListContext ctx) {
        List<DfmValue> values = new ArrayList<>();
        for (var child : ctx.children) {
            String text = child.getText();
            if (text.equals("(") || text.equals(")"))
                continue;
            if (child instanceof TerminalNode tn) {
                int tokenType = tn.getSymbol().getType();
                if (tokenType == DelphiDFMParser.INT) {
                    values.add(new DfmValue(DfmValue.Type.INTEGER, Integer.parseInt(text)));
                } else if (tokenType == DelphiDFMParser.FLOAT) {
                    values.add(new DfmValue(DfmValue.Type.FLOAT, Double.parseDouble(text)));
                } else if (tokenType == DelphiDFMParser.STRING) {
                    values.add(new DfmValue(DfmValue.Type.STRING, text.substring(1, text.length() - 1)));
                }
            }
        }
        return new DfmValue(DfmValue.Type.PAREN_LIST, values);
    }
}
```

### Conclusion

Now we have everything in place to wire it all up to a `Main` class, and generate our *AST*.