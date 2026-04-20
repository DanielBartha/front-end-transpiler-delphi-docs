# Final Cleanup and Generation

Now that we finished our implementation, we'll need to re-compile all our java files. 

**IMPORTANT:** make sure you save all your custom java files in a separate directory first !

After all is saved, we clean up potentially stale files:

1. Remove potentially stale files

```
rm *.java *.class *.tokens *.interp
```

2. Copy and paste your custom java files back to root

3. Re-generate the parser with the visitor command

```
antlr4 DelphiDFM.g4 -visitor
```

4. Compile everything

```
javac *.java
```

5. Run java on Main and see the magic unfold

```
java Main DFMs/test.dfm
```

Let's examine the fruits of our labor:

`../Generated-ASTs/testAST.dfm`
```yaml
[ROOT | inherited | VISUAL_CONTAINER] frmActions : TfrmActions
 [CONFIG] Tag = 3
 [VISUAL] Caption = 'hello'
  [inherited | VISUAL_CONTAINER] pnlTitle : TfrmTitleBar
   [VISUAL] Color = clWhite
   [VISUAL] ParentBackground = False
  [inherited | VISUAL_CONTAINER] pnlError : TfrmError
    [inherited | VISUAL_CONTROL] lblMessage : TcxLabel
     [LAYOUT] AnchorY = -18
     [CONFIG] FloatAnchorY = 18.7283
  [object | VISUAL_CONTROL] gvListUser : TcxGridDBColumn
   [VISUAL] Caption = 'Test'
   [DATA_BINDING] DataBinding.FieldName = 'TestUser'
   [CONFIG_NAMESPACED] Properties.ListColumns = ItemList{
  item[0]:
    [CONFIG] FieldName = 'fullName'
    [CONFIG] Fixed = True
    [LAYOUT] Width = 100
}
   [CONFIG_NAMESPACED] Properties.Alignment.Vert = taVCencter
   [LAYOUT] Height = 20
   [CONFIG_NAMESPACED] Properties.DateButtons = [btnToday]
  [object | null] testItemHchy : TestItemHchy
   [CONFIG_NAMESPACED] Properties.ListColumns = ItemList{
  item[0]:
    [CONFIG] Fixed = True
    [LAYOUT] Width = 80
  item[1]:
    [LAYOUT] Width = 200
    [CONFIG] FieldName = 'testing'
}
   [LAYOUT] Width = 282
  [inherited | null] testFields : TestFields
   [CONFIG] Indexes = ItemList{
}
   [CONFIG] SortOptions = []
   [CONFIG] OtherSortOptions = [test1, test2, test3]
  [inherited | null] testMultipleInheritance : TestMultipleInheritance
    [inherited | null] inhOne : InhOne
      [inherited | null] inhTwo : InhTwo
        [inherited | null] inhThree : InhThree
          [object | null] deepObj : DeepObj
           [LAYOUT] Height = 10
           [LAYOUT] Width = 20
           [LAYOUT] Top = 30
```

We successfully generated an *Abstract Syntax Tree* from our `test.dfm` file.

- we have our root component distinctly specified
- components have the their correct roles assigned
- property categories correctly appear
- we have our collection item lists and their properties
- some roles and categories are `null`, since the pseudo-code examples haven't been classified in `PropertyCategory` and `ComponentRole`

## Conclusion

Now we have successfully paved the way to designing the transformation rules for the transpiler's next component.