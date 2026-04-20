# Property and Component Classifications

The last pre-emptive step before hooking up everything to `ASTBuilder` and `Main`, will be to classify properties and components for our *AST* nodes. By having a more detailed array of notations, we'll be able to formulate transformation rules more easily.

## Property Categories

The following 8 node categories have been defined for properties:

`PropertyCategoryEnum.java`
```java
public enum PropertyCategoryEnum {
    LAYOUT,
    VISUAL,
    CONFIG,
    CONFIG_NAMESPACED,
    DATA_BINDING,
    EVENT,
    REFERENCE,
    SKIP
}
```

**LAYOUT** properties are self-explanatory. They control visual sizing and positions. In React, they should map directly to CSS style objects.
Examples of this category include: `Left`, `Top`, `Width`, `MinSize`.

The **VISUAL** category refers to *how* a component looks, but not its position. In React, they should map to CSS styling or component props.
Some examples include: `Color`, `ParentColor`, `Transparent`, `Caption`. 

**CONFIG** is the largest category by far. This category represents properties which configure how a component behaves at runtime, but do not affect layout or appearance. The typical transition to React would be as component props. Some examples are: `TabOrder`, `AutoSize`, `SortOrder`, `FieldName`.

**CONFIG_NAMESPACED** properties are configurations scoped under a sub-object. Syntactically, they fall under our grammar's `SUBPROPERTY` rule. Semantically, they are still configuration but grouped. Examples: `Options.Filtering`, `OptionsData.Editing`, `Properties.Alignment.Vert`.

The **DATA_BINDING** category refers to connections between a UI component and a data source. These have direct implications for React state management. Examples include: `DataBinding.FieldName`, `DataController.DataSource`, `DataSet`.

The **EVENT** category refers to component that wire up user interactions or lifecycle events to Object Pascal methods. In React, they will become callback props or handler functions. Some examples are: `OnClick`, `OnChange`, `BeforePost`.

The **REFERENCE** category encompasses components which point to other components, either within the same file or in an external module. Some examples include: `Control = pnlEntry`, `GridView = gvList`, `Action = frmMain.aiAdd`. 

Finally, the **SKIP** category encompasses design-time properties which are irrelevant to web, as such they will be skipped during transformation. An example of this property is `PixelsPerInch`.

### Property Classifier

The `PropertyClassifier` makes use of the categories we defined above, to assign the proper notations.

Since the **CONFIG** category is very large (when accounting for the full legacy codebase), the most suitable solution is to combine a fallback logic with hash maps. We start out with instantiating the map, and diving into the fallback logic right away:

```java
public class PropertyClassifier {
    private static final Map<String, PropertyCategoryEnum> PROPERTY_MAP = new HashMap<>();

    public static PropertyCategoryEnum classify(String propertyName) {
        if (propertyName.startsWith("On") 
            || propertyName.startsWith("Before") 
            || propertyName.startsWith("After")) {
            return PropertyCategoryEnum.EVENT;
        }

        PropertyCategoryEnum category = PROPERTY_MAP.get(propertyName);
        if (category != null) {
            return category;
        }
    
        if (propertyName.contains(".")) {
            return classifyNamespaced(propertyName);
        }

        return PropertyCategoryEnum.CONFIG;
    }

    ...

}
```

We tackle events first, based on the keywords `On`, `Before` and `After`. We make sure to return the category, and also account for namespaced properties (more on that soon). Finally, we if all else fails, we fall back to **CONFIG**.

Continuing, with namespaced properties:

```java
public class PropertyClassifier {
    private static final Map<String, PropertyCategoryEnum> PROPERTY_MAP = new HashMap<>();

    ...

    private static PropertyCategoryEnum classifyNamespaced(String propertyName) {
        PropertyCategoryEnum category = PROPERTY_MAP.get(propertyName);
        if (category != null) {
            return category;
        }
    
        String lastSegment = propertyName.substring(propertyName.lastIndexOf('.') + 1);
        if (lastSegment.startsWith("On") 
            || lastSegment.startsWith("Before") 
            || lastSegment.startsWith("After")) {
            return PropertyCategoryEnum.EVENT;
        }
    
        if (propertyName.startsWith("DataBinding.") 
            || propertyName.equals("DataController.DataSource")) {
            return PropertyCategoryEnum.DATA_BINDING;
        }

        return PropertyCategoryEnum.CONFIG_NAMESPACED;
    }

    ...

}
```

After returning the category, we make sure to account for namespaced events as well (for instance: `Styles.OnGetContentStyle`). Then we tackle data binding properties, and fall back to base **CONFIG_NAMESPACED**.

Our last step is to insert all other outlying values in our hash map:

```java
public class PropertyClassifier {
    private static final Map<String, PropertyCategoryEnum> PROPERTY_MAP = new HashMap<>();

    ...

    static {
        // Layout Properties
        PROPERTY_MAP.put("Left", PropertyCategoryEnum.LAYOUT);
        PROPERTY_MAP.put("Top", PropertyCategoryEnum.LAYOUT);
        PROPERTY_MAP.put("Width", PropertyCategoryEnum.LAYOUT);
        ...

        // Visual Properties
        PROPERTY_MAP.put("Color", PropertyCategoryEnum.VISUAL);
        PROPERTY_MAP.put("ParentColor", PropertyCategoryEnum.VISUAL);
        PROPERTY_MAP.put("ParentBackground", PropertyCategoryEnum.VISUAL);
        ...

        // Cross Component Refs
        PROPERTY_MAP.put("FocusControl", PropertyCategoryEnum.REFERENCE);
        PROPERTY_MAP.put("PopupMenu", PropertyCategoryEnum.REFERENCE);
        PROPERTY_MAP.put("Control", PropertyCategoryEnum.REFERENCE);
        ...

        // Data Binding
        PROPERTY_MAP.put("DataSet", PropertyCategoryEnum.DATA_BINDING);

        // Skip, irrelevant to web
        PROPERTY_MAP.put("PixelsPerInch", PropertyCategoryEnum.SKIP);
    }
}
```

**Note** the outlying data binding property `DataSet`.

Now, we fully covered property classification, and we are ready to move on to component roles.

## Component Role Categories

This classification is used as a helper containerization of properties within DFM objects. By specifying the roles of components, we'll be able to further streamline the design of transformation rules. It is considered a necessary addition, which highlights an important nuance of certain DFM components: the same property can mean different things on different component types. 

For instance, `Background` under a `TfrxPDFExport` object is **CONFIG**, as it's a boolean flag for PDF generation, not **VISUAL**. However, `Background` under `TfrxHTMLExport` is also **CONFIG**. Neither produces visual output. Consequently, `Background` will be classified under **CONFIG** everywhere else as well, but if it appears under a *visual* component, it might be evaluated as **VISUAL**.

The following 5 categories have been defined as component roles:

`ComponentRoleEnum.java`
```java
public enum ComponentRoleEnum {
    VISUAL_CONTAINER,
    VISUAL_CONTROL,
    DATA_FIELD,
    NON_VISUAL_DATA,
    NON_VISUAL_SERVICE
}
```

**VISUAL_CONTAINER** denotes components that contain visual properties. Note that the classification of roles is only a suggestive helper addition, as properties within **VISUAL_CONTAINER** components could classify under various other property categories. Examples of such roles include: `TPanel`, `TcxGrid`, `TfrmActions`.

**VISUAL_CONTROL** roles refer to leaf UI components that the user directly interacts with or observes. These components do not primarily serve as containers for other components, but rather represent individual interface elements such as buttons, labels, input fields, and grid columns. For instance:

```yaml
inherited pnlEntry: TPanel
    inherited psActionButton1: TcxButton
      TabOrder = 7
    end
end
```

Could be labeled the following way in our generated *AST*:

```yaml
[inherited | VISUAL_CONTAINER] pnlEntry : TPanel
    [inherited | VISUAL_CONTROL] psActionButton1 : TcxButton
     [CONFIG] TabOrder = 7
```

The **DATA_FIELD** role specifies field definitions within data components. These are children of **NON_VISUAL_DATA** components such as `TdxMemData`, and define the shape of the data structure.

```yaml
inherited mdList: TdxMemData
    object mdListInternalID: TIntegerField
      FieldName = 'InternalID'
    end
```

Would become...

```yaml
[inherited | NON_VISUAL_DATA] mdList : TdxMemData
    [object | DATA_FIELD] mdListInternalID : TIntegerField
     [CONFIG] FieldName = 'InternalID'
```

While **NON_VISUAL_DATA** is a self-explanatory rule, we shall briefly elaborate the final component role category: **NON_VISUAL_SERVICE**. This role denotes objects whose functionality cannot, or should not be transpiled to React. These mostly include features such as report/document generation, usually hailing from `Tfrx` type objects. These types of object *will* be transpiled, but only for the purpose of conveying information to developers, as some of these could hold relevant data, helpful to developers during the modernization process. The transpiled versions of these objects will not be functional, but they will inform developers on stylistic choices, contained custom information, and placement within architecture.

### Component Role Classifier

The `ComponentClassifier` is more straightforward. The approach is similar, we create a hash map and insert our various component roles, based on our previously defined categories:

```java
public class ComponentClassifier {
    private static final Map<String, ComponentRoleEnum> ROLE_MAP = new HashMap<>();

    static {
        // Visual Containers
        ROLE_MAP.put("TPanel", ComponentRoleEnum.VISUAL_CONTAINER);
        ROLE_MAP.put("TcxGrid", ComponentRoleEnum.VISUAL_CONTAINER);
        ...

        // Visual Controls
        ROLE_MAP.put("TcxButton", ComponentRoleEnum.VISUAL_CONTROL);
        ROLE_MAP.put("TcxLabel", ComponentRoleEnum.VISUAL_CONTROL);
        ROLE_MAP.put("TcxDBMemo", ComponentRoleEnum.VISUAL_CONTROL);
        ...

        // Data Fields
        ROLE_MAP.put("TIntegerField", ComponentRoleEnum.DATA_FIELD);
        ROLE_MAP.put("TStringField", ComponentRoleEnum.DATA_FIELD);
        ROLE_MAP.put("TDateField", ComponentRoleEnum.DATA_FIELD);
        ...

        // Non-Visual Data
        ROLE_MAP.put("TDataSource", ComponentRoleEnum.NON_VISUAL_DATA);
        ROLE_MAP.put("TdxMemData", ComponentRoleEnum.NON_VISUAL_DATA);

        // Non-Visual Service
        ROLE_MAP.put("TfrxReport", ComponentRoleEnum.NON_VISUAL_SERVICE);
        ROLE_MAP.put("TfrxPDFExport", ComponentRoleEnum.NON_VISUAL_SERVICE);
        ROLE_MAP.put("TfrxCSVExport", ComponentRoleEnum.NON_VISUAL_SERVICE);
        ...
    }

    public static ComponentRoleEnum classify(String typeName) {
        return ROLE_MAP.get(typeName);
    }
}
```

At the end we return the component type name, so that we can hook it up to our `ASTBuilder`.

#### Conclusion

Now we are ready to move into the final two steps of *AST* generation, by hooking up everything to an `ASTBuilder` class and implementing *ANTLR's* visitor pattern when processing our target DFM files.