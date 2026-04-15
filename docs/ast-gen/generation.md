# Regenerate the Parser

Now that we have updated our grammar, we will need to regenerate out parser. This involves the standard process we have followed until now, however there is an important distinction. In order for *ANTLR* to use our annotations, we will need to make use of *ANTLR's* tree walking feature by generating the parser with an additional command.

Let's see the process:

1. Clean up old files

```rm *.java *.class *.tokens *.interp```

2. Regenerate the parser with *ANTLR's* tree walking feature

```antlr4 DelphiDFM.g4 -visitor```

By including the `-visitor` command, we tell *ANTLR* to generate the *DelphiDFMBaseVisitor* and *DelphiDFMVisitor* classes, which will include methods based on our annotations. For instance:

`../DelphiDFMVisitor.java`
```java
...

public interface DelphiDFMVisitor<T> extends ParseTreeVisitor<T> {
	/**
	 * Visit a parse tree produced by {@link DelphiDFMParser#file}.
	 * @param ctx the parse tree
	 * @return the visitor result
	 */
	T visitFile(DelphiDFMParser.FileContext ctx);
	/**
	 * Visit a parse tree produced by the {@code beginBlock}
	 * labeled alternative in {@link DelphiDFMParser#block}.
	 * @param ctx the parse tree
	 * @return the visitor result
	 */
	T visitBeginBlock(DelphiDFMParser.BeginBlockContext ctx);
	/**
	 * Visit a parse tree produced by the {@code subProperty}
	 * labeled alternative in {@link DelphiDFMParser#propertyBlock}.
	 * @param ctx the parse tree
	 * @return the visitor result
	 */

     ...
}
```

and

`../DelphiDFMBaseVisitor.java`
```java
...

public class DelphiDFMBaseVisitor<T> extends AbstractParseTreeVisitor<T> implements DelphiDFMVisitor<T> {
	/**
	 * {@inheritDoc}
	 *
	 * <p>The default implementation returns the result of calling
	 * {@link #visitChildren} on {@code ctx}.</p>
	 */
	@Override public T visitFile(DelphiDFMParser.FileContext ctx) { return visitChildren(ctx); }
	/**
	 * {@inheritDoc}
	 *
	 * <p>The default implementation returns the result of calling
	 * {@link #visitChildren} on {@code ctx}.</p>
	 */
	@Override public T visitBeginBlock(DelphiDFMParser.BeginBlockContext ctx) { return visitChildren(ctx); }
	/**
	 * {@inheritDoc}
	 *
	 * <p>The default implementation returns the result of calling
	 * {@link #visitChildren} on {@code ctx}.</p>
	 */

    ...
}
```

Of course, we can also find them within our generated parser, where the real logic of vistors lives:

`../DelphiDFMParser.java`
```java
    ...

	@SuppressWarnings("CheckReturnValue")
	public static class BlockContext extends ParserRuleContext {
		public BlockContext(ParserRuleContext parent, int invokingState) {
			super(parent, invokingState);
		}
		@Override public int getRuleIndex() { return RULE_block; }
	 
		public BlockContext() { }
		public void copyFrom(BlockContext ctx) {
			super.copyFrom(ctx);
		}
	}
	@SuppressWarnings("CheckReturnValue")
	public static class BeginBlockContext extends BlockContext {
		public List<TerminalNode> ID() { return getTokens(DelphiDFMParser.ID); }
		public TerminalNode ID(int i) {
			return getToken(DelphiDFMParser.ID, i);
		}
		public TerminalNode END() { return getToken(DelphiDFMParser.END, 0); }
		public TerminalNode INHERITED() { return getToken(DelphiDFMParser.INHERITED, 0); }
		public TerminalNode OBJECT() { return getToken(DelphiDFMParser.OBJECT, 0); }
		public TerminalNode INLINE() { return getToken(DelphiDFMParser.INLINE, 0); }
		public List<PropertyBlockContext> propertyBlock() {
			return getRuleContexts(PropertyBlockContext.class);
		}
		public PropertyBlockContext propertyBlock(int i) {
			return getRuleContext(PropertyBlockContext.class,i);
		}
		public List<BlockContext> block() {
			return getRuleContexts(BlockContext.class);
		}
		public BlockContext block(int i) {
			return getRuleContext(BlockContext.class,i);
		}
		public BeginBlockContext(BlockContext ctx) { copyFrom(ctx); }
		@Override
		public void enterRule(ParseTreeListener listener) {
			if ( listener instanceof DelphiDFMListener ) ((DelphiDFMListener)listener).enterBeginBlock(this);
		}
		@Override
		public void exitRule(ParseTreeListener listener) {
			if ( listener instanceof DelphiDFMListener ) ((DelphiDFMListener)listener).exitBeginBlock(this);
		}
		@Override
		public <T> T accept(ParseTreeVisitor<? extends T> visitor) {
			if ( visitor instanceof DelphiDFMVisitor ) return ((DelphiDFMVisitor<? extends T>)visitor).visitBeginBlock(this);
			else return visitor.visitChildren(this);
		}
	}

    ...
```

3. Compile everything

```javac *.java ```

Everything should compile at this point, and we should be ready to implement the *AST* generation.
