# Wiring Up to Main

Wiring everything up to `Main` will be a straightforward process. And because we are fancy, we'll save the generated *ASTs* as a text file to a custom directory.

We start by creating our `main` class. This class is responsible for calling the `processDFM()` function, into which we pass command line arguments. This is how we will generate the *ASTs*. In this implementation we create a directory called *DFMs*, into which we store the `.dfm` file we wish to process.

```java
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.*;

import java.io.IOException;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.Writer;
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {
        if (args.length > 0) {
            processDFM(args[0]);
        } else {
            File dfmDir = new File("DFMs");
            
            if (!dfmDir.exists() || !dfmDir.isDirectory()) {
                System.out.println("DFMs directory not found");
                return;
            }
            
            File[] dfmFiles = dfmDir.listFiles((dir, name) -> name.endsWith(".dfm"));
            
            if (dfmFiles == null || dfmFiles.length == 0) {
                System.out.println("No .dfm files found in DFMs directory");
                return;
            }
            
            System.out.println("Processing " + dfmFiles.length + " .dfm files...\n");
            for (File dfmFile : dfmFiles) {
                processDFM(dfmFile.getAbsolutePath());
            }
            System.out.println("All files processed successfully");
        }
    }

    ...

}
```

Next, we implement the `processDFM()` function:

```java
public class Main {

    ...

    private static void processDFM(String filePath) throws IOException {
            try {
                CharStream input = CharStreams.fromFileName(filePath);

                DelphiDFMLexer lexer = new DelphiDFMLexer(input);
                CommonTokenStream tokens = new CommonTokenStream(lexer);

                DelphiDFMParser parser = new DelphiDFMParser(tokens);
                ParseTree tree = parser.file();

                ASTBuilder builder = new ASTBuilder();
                roots = (List<ComponentNode>) builder.visit(tree);

                saveAST(filePath);

            } catch (Exception e) {
                System.err.println("Error processing file: " + filePath);
                e.printStackTrace();
            }
        }

        private static List<ComponentNode> roots;

        static boolean isRoot(ComponentNode node) {
            if (roots.indexOf(node) == 0) {
                return true;
            }
            return false;
        }

        ...

}
```

Here, we pass the input with `CharStreams`, then instantiate the lexer, parser, and `ASTBuilder`. We visit the fully populated `roots` list, and call a function to save our *AST* (more on that shortly). We also catch any potential errors.

There is a small peculiarity with `roots` here. We exclude it from the main function, so that we can determine whether the parsed component is on root level. We use this information later during printing, for the purpose of adding an additional component role, which specifies which component acts as the root.

Next, we create the `printComponent` function, where we use `isRoot` to differentiate from non-root components; we additionally print the property nodes, and recursively print component node children.

```java
public class Main {

    ...

    static void printComponent(ComponentNode node, int depth, Writer writer) throws IOException {
        String indent = "  ".repeat(depth);

        if (isRoot(node)) {
            writer.write(indent + "[" + "ROOT" + " | " + node.keyword + " | " + node.role + "] " + node.name + " : " + node.typeName + "\n");
        } else {
            writer.write(indent + "[" + node.keyword + " | " + node.role + "] " + node.name + " : " + node.typeName + "\n");
        }

        for (PropertyNode p : node.properties) {
            writer.write(indent + " [" + p.category + "] " + p.name + " = " + p.value + "\n");
        }

        for (ComponentNode child : node.children) {
            printComponent(child, depth + 1, writer);
        }
    }

    ...

}
```

The final part consists of implementing `saveAST()`:

```java
public class Main {

    ...

        public static void saveAST(String inputFile) {
        File fileDir = new File("Generated-ASTs");
        String fileName = new File(inputFile).getName();
        String baseName = fileName.substring(0, fileName.lastIndexOf('.'));
        String outputFile = baseName + "AST" + ".txt";

        if (!fileDir.exists()) {
            if(fileDir.mkdirs()){
                System.out.println("directory for ASTs created");
            } else {
                System.out.println("AST folder creation failed");
            }
        } else {
            System.out.println("directory for ASTs already exists");
        }

        File outFile = new File(fileDir, outputFile);
        
        try {
            try (BufferedWriter save = new BufferedWriter (new FileWriter(outFile))) {
                for (ComponentNode root : roots) {
                    printComponent(root, 0, save);
                }
                System.out.println(outputFile + " saved to Generated-ASTs directory");
            } 
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

This implementation is fairly stratghtforward:

- create a new directory for the *ASTs*
- pass the target file name and modify it to distinguish between the original and the *AST* generated from it
- check if directory exists, and notify the user
- use `printComponent()` to populate the new file
- and catch any errors

## Conclusion

And we are finished. However, before we can actually generate anything, we'll need to take a couple of steps. The following (and final) page of this chapter will walk you through the last steps of generating an *AST*.