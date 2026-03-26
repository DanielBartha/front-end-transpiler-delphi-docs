# DFM Parsing with ANTLR4

Now that the setup is complete, the time has come to parse some DFM code.

First things first, let's copy our DFM grammar into a fresh directory, included in a file with the `.g4` extension.

**Notice**: remember that the file name has to correspond to the grammar specification in the `.g4` file.

```../dir/DelphiDFM.g4``` <=> ```grammar DelphiDFM ;```

Next, we generate the lexer and parser using *ANTLR*:

```antlr4 DelphiDFM.g4```

Output:

```bash
ls

DelphiDFMBaseListener.java  
DelphiDFMLexer.interp  
DelphiDFMListener.java
DelphiDFM.g4                
DelphiDFMLexer.java    
DelphiDFMParser.java
DelphiDFM.interp            
DelphiDFMLexer.tokens  
DelphiDFM.tokens
```

Great! The next step is to compile all `.java` files generated, like so:

```javac *.java```

Now we get the compiled files:

```bash
ls

DelphiDFMBaseListener.class  
DelphiDFMBaseListener.java   
DelphiDFM.g4                 
DelphiDFM.interp             
DelphiDFMLexer.class         
DelphiDFMLexer.interp        
DelphiDFMLexer.java          
DelphiDFMLexer.tokens        
DelphiDFMListener.class      
DelphiDFMListener.java
'DelphiDFMParser$BlockContext.class'
'DelphiDFMParser$FileContext.class'
'DelphiDFMParser$ItemContext.class'
'DelphiDFMParser$MethodLSContext.class'
'DelphiDFMParser$PropertyBlockContext.class'
'DelphiDFMParser$ValueContext.class'
 DelphiDFMParser.class
 DelphiDFMParser.java
 DelphiDFM.tokens
```

Aha! Now we can see our parser rules as well.

<img src="https://media1.tenor.com/m/sqYV7D2euF8AAAAd/kronk-oh-yeah-its-all-coming-together.gif" alt="funny gif" width="300"/>


## Test Against a DFM File

The only thing we are missing is a test file that we should parse. The DFM file below was conceived to test all rules we previously defined in our grammar:

`../dir/test.dfm`
```ini
inherited frmActions: TfrmActions
    Tag = 3
    Caption = 'hello'
    inherited pnlTitle: TfrmTitleBar
        Color = clWhite
        ParentBackground = False
    end
    inherited pnlError: TfrmError
        inherited lblMessage: TcxLabel
            AnchorY = -18
            FloatAnchorY = 18.7283
        end
    end
    object gvListUser: TcxGridDBColumn
        Caption = 'Test'
        DataBinding.FieldName = 'TestUser'
        Properties.ListColumns = <
            item
                FieldName = 'fullName'
                Fixed = True
                Width = 100
            end>
        Properties.Alignment.Vert = taVCencter
        Height = 20
        Properties.DateButtons = [btnToday]
    end
    object testItemHchy: TestItemHchy
        Properties.ListColumns = <
        item
            Fixed = True
            Width = 80
        end
        item
            Width = 200
            FieldName = 'testing'
        end>
        Width = 282
    end
    inherited testFields: TestFields
        Indexes = <>
        SortOptions = []
        OtherSortOptions = [test1, test2, test3]
    end
    inherited testMultipleInheritance: TestMultipleInheritance
        inherited inhOne: InhOne
            inherited inhTwo: InhTwo
                inherited inhThree: InhThree
                    object deepObj: DeepObj
                        Height = 10
                        Width = 20
                        Top = 30
                    end
                end
            end
        end
    end
end
```

## Token Testing

Now we have all necessary components. First, let's take a look at all identified tokens from our `test.dfm` file. We use `grun`, which is our alias for *ANTLR's* testing rig: `org.antlr.v4.gui.TestRig`.

We use the command:

```grun DelphiDFM file -tokens test.dfm```

And our output:

```bash
[@0,0:8='inherited',<'inherited'>,1:0]
[@1,10:19='frmActions',<ID>,1:10]
[@2,20:20=':',<':'>,1:20]
[@3,22:32='TfrmActions',<ID>,1:22]
[@4,38:40='Tag',<ID>,2:4]
[@5,42:42='=',<'='>,2:8]
[@6,44:44='3',<INT>,2:10]
[@7,50:56='Caption',<ID>,3:4]
[@8,58:58='=',<'='>,3:12]
[@9,60:66=''hello'',<STRING>,3:14]
[@10,72:80='inherited',<'inherited'>,4:4]
[@11,82:89='pnlTitle',<ID>,4:14]
[@12,90:90=':',<':'>,4:22]
[@13,92:103='TfrmTitleBar',<ID>,4:24]
[@14,113:117='Color',<ID>,5:8]
[@15,119:119='=',<'='>,5:14]
[@16,121:127='clWhite',<ID>,5:16]
[@17,137:152='ParentBackground',<ID>,6:8]
[@18,154:154='=',<'='>,6:25]
[@19,156:160='False',<ID>,6:27]
[@20,166:168='end',<'end'>,7:4]
[@21,174:182='inherited',<'inherited'>,8:4]
[@22,184:191='pnlError',<ID>,8:14]
[@23,192:192=':',<':'>,8:22]
[@24,194:202='TfrmError',<ID>,8:24]
[@25,212:220='inherited',<'inherited'>,9:8]
[@26,222:231='lblMessage',<ID>,9:18]
[@27,232:232=':',<':'>,9:28]
[@28,234:241='TcxLabel',<ID>,9:30]
[@29,255:261='AnchorY',<ID>,10:12]
[@30,263:263='=',<'='>,10:20]
[@31,265:267='-18',<INT>,10:22]
[@32,281:292='FloatAnchorY',<ID>,11:12]
[@33,294:294='=',<'='>,11:25]
[@34,296:302='18.7283',<FLOAT>,11:27]
[@35,312:314='end',<'end'>,12:8]

...

```
Continued further until all tokens are consumed.

**Note** that in our given command, `file` specifies the rule that needs to contain our input.

### Token Decomposition

Now we can see an output of all identified tokens from our test file. Let's decompose a token so we understand how they are built. For instance, our second token:

```[@1,10:19='frmActions',<ID>,1:10]```

Each line of the output represents a single token and shows everything we know about the token: 

- `@1` indicates that it is the second token (indexed from 0).
- `10:19=` indicates that it goes from character position 10 to 19 (inclusive starting from 0).
- `'frmActions'` indicates what text it has.
- `<ID>` indicates the type of token.
- finally, `1:10` indicates that it is on line 1 (from 1), and is at character position 10 (starting from zero and counting tabs as a single character).

In the next section, we will take a look at outputting a parse tree, based on our identified tokens. **Spoiler:** we will also visualize them.