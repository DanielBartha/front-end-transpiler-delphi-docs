# Introduction

The design of transformation rules (and later on the transformation model) is regarded as the most complex component of this tool. It is a crucial step, which necessitates careful architecting to ensure a valid output.

The transformation rules are comprised of a series of design decisions that specify how the conversion from source *AST* to target *AST* should be executed. This encapsulates the core component of the **Transformation Model**, regarded as the **Semantic Transformation Layer** within the transpiler pipeline.

## Preparations

In order to transpile a DFM file to React, the first step is to identify all dependencies of the chosen file. Determining the inheritance hierarchy of a file is a crucial step in establishing the transformation sequence.

Given a 5-level deep inheritance hierarchy of a chosen subset, we shall establish the philosophy of designing transformations for inherited non-root components. The chosen subset's hierarchy has been traced in the figure below, and is used as reference throughout the process of designing transformation rules:

![Class Diagram of ActionsFrame.dfm Tracing its Inheritance Hierarchy](assets/dfm-composition-diagram.png)

The transpiling strategy will follow a bottom-up approach, working upwards from the furthest leaf-node: `ActionsFrame.dfm`. This will mean that stubs will be created for missing functionality as the inheritance hierarchy gets transpiled sequentially.

### AST Node Refinement

The first step we'll need to take, is to properly define the node categories:

| Node Categories | Examples |  |
|-------------------|----------|----------|
| **Layout Properties**<br>These control visual positioning and sizing. <br>They map directly to CSS style objects in React. | • Left<br>• Top<br>• Width<br>• Height<br>• Align<br>• Anchors<br> |• MinWidth<br> •BestFitMaxWidth<br>• MinSize<br>• DesignSize<br>• ExplicitWidth
| **Visual/Appearance Properties** <br>These control how a component looks but are not about position. <br>Will map to CSS styling or component props in React. | • Color<br>• ParentColor<br>• ParentBackground<br>• Visible<br>• Caption<br>|• Transparent<br>• BevelOuter<br>• PaintStyle<br>• BorderWidth |
| **Behavioral/Configuration Properties**<br>These configure how the component behaves at runtime but do not directly affect layout or appearance. <br>Typical transition for React: component props. | • TabOrder<br>• TabStop<br>• Enabled<br>• AutoSize<br>• SortIndex<br>• SortOrder<br>• SortOptions<br>• GroupIndex<br>|• DisplayWidth<br>• Size<br>• FieldName<br>• Version<br>• DotMatrixReport<br>• Separator<br>• UTF8 |
| **Namespaced Configuration Properties**<br>These are configuration properties scoped under a sub object. Syntactically they use the `SUBPROPERTY` token. Semantically they are still configuration but grouped. | • Options.Filtering<br>• OptionsData.Editing<br>• OptionsView.CellEndEllipsis<br>• Properties.Alignment.Vert<br>• Properties.WordWrap<br>• SpeedButtonOptions.Transparent<br>|• Constraints.MaxHeight<br>• HotZone.SizePercent<br>• PreviewOptions.Zoom<br>• PrintOptions.Printer<br> •ReportOptions.CreateDate |
| **Data Binding Properties**<br>These connect a UI component to a data source. They have direct implications for React state management. | • DataBinding.FieldName<br>• DataBinding.DataField<br>• DataBinding.DataSource<br>| •DataController.DataSource<br>• DataSet |
| **Event Handlers**<br>These wire up user interactions or lifecycle events to Object Pascal methods. <br>In React, they will become callback props or handler functions. | • OnClick<br>• OnChange<br>• OnTimer<br>• OnEnter<br>• OnExit<br>• OnResize<br>• OnDblClick<br>• OnGetDisplayText<br>|• OnSendMail<br>• OnCheckEOF<br>• OnFirst<br>• OnNext<br>• OnPrior<br>• OnGetValue<br>• BeforePost |
| **Cross Component References**<br>These point to another component either within the same file or in an external module. | • FocusControl = edDate<br>• PopupMenu = popList<br>• Control = pnlEntry<br>• GridView = gvList<br>• Properties.ListSource = dsUser<br>|• Action = frmMain.aiAdd<br>• OptionsImage.Images = dmMain.ilNormal16x16<br>• BarManager = frmMain.BarManager |
| **Non Visual Service Components**<br>These are entire component declarations (not individual properties) that have no visual representation. Such as data sources, timers, report generators, exporters, and so on. | • TDataSource<br>• TdxMemData<br>• TfrxReport<br>• TfrxPDFExport<br>• TfrxCSVExport<br>• TfrxHTMLExport<br>• TfrxSimpleTextExport<br>• TfrxMailExport<br>|• TfrxUserDataSet<br>• TTimer<br>• TdxBarPopupMenu<br>• TIntegerField<br>• TStringField<br>• TDateField<br>• TFloatField<br>• TDateTimeField |

