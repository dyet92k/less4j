Less language is an extension of css and this project compiles it into regular css. It adds several dynamic features into css: variables, expressions, nested rules, and so on. Our [documentation](https://github.com/SomMeri/less4j/wiki) contains both overview of all these features and links to their detailed descriptions. If you are interested in the language itself, we recommend to start with the [overview](https://github.com/SomMeri/less4j/wiki/Supported-Less-Language).   

The original compiler was written in JavaScript and is called [less.js](http://lesscss.org/). Official less is mostly defined in less.js documentation/issues and by what less.js actually do. Links to less.js:
* [less.js home page](http://lesscss.org/) 
* [less.js source code & issues](https://github.com/cloudhead/less.js) 

Less4j is a port and its behavior should be as close to the original implementation as reasonable. Unless explicitly stated otherwise, any difference between less.js and less4j outputs is considered a bug. As close as reasonable means that style sheets generated by less4j must be functionally the same as the outputs of less.js. However, they do not have to be exactly the same:
* Behavior of less.js and less4j may differ on invalid input files.
* Output files may differ by whitespaces or comments locations.
* Less4j may do more than less.js in some situations. The input rejected by less.js may be accepted and translated by less4j. 

All known differences are documented on [wiki page](https://github.com/SomMeri/less4j/wiki/Differences-Between-Less.js-and-Less4j). Less4j also produces warnings any time it produces functionally different CSS.

## Continuous Integration
Continuous integration is set up on [Travis-CI](http://travis-ci.org/SomMeri/less4j), its current status is: [![Build Status](https://secure.travis-ci.org/SomMeri/less4j.png)](http://travis-ci.org/SomMeri/less4j).

## Twitter
Our twitter account: [Less4j](https://twitter.com/Less4j)

## Documentation:
The documentation is kept on Github wiki:
* [wiki home page](https://github.com/SomMeri/less4j/wiki),
* [all written wiki pages](https://github.com/SomMeri/less4j/wiki/_pages). 

For those interested about project internals, architecture and comments handling are described in a [blog post] (http://meri-stuff.blogspot.sk/2012/09/tackling-comments-in-antlr-compiler.html). The blog post captures our ideas at the time of its writing, so current implementation may be a bit different.

## Command Line
Less4j can run from [command line](https://github.com/SomMeri/less4j/wiki/Command-Line-Options). Latest versions are shared via [less4j dropbox account](https://www.dropbox.com/sh/zcb8p27db9ou4x1/keQWIZziH8). Shared folder always contains at least two latest versions, but we may remove older ones. 

If you need an old version for some reason, checkout appropriate tag from git and use `mvn package -P standalone` command. The command compiles less4j and all its dependencies into `target/less4j-<version>-shaded.jar` file. 

## Maven
Less4j is [available](http://search.maven.org/#browse|1893223923) in Maven central repository.

Pom.xml dependency:
<pre><code>&lt;dependency&gt;
  &lt;groupId&gt;com.github.sommeri&lt;/groupId&gt;
  &lt;artifactId&gt;less4j&lt;/artifactId&gt;
  &lt;version&gt;1.4.1&lt;/version&gt;
&lt;/dependency&gt;
</code></pre>

## Integration With Wro4j
The easiest way to integrate less4j into Java project is to use [wro4j](http://alexo.github.com/wro4j/) library. More about wro4j can be found either in a [blog post](http://meri-stuff.blogspot.sk/2012/08/wro4j-page-load-optimization-and-lessjs.html) or on wro4j [google code](http://code.google.com/p/wro4j/) page.

## API Description
Access the compiler through the `com.github.sommeri.less4j.LessCompiler` interface. Its thread safe implementation is `com.github.sommeri.less4j.core.DefaultLessCompiler`. The interface exposes following methods:
*  `CompilationResult compile(File inputFile)` - compiles a file, 
*  `CompilationResult compile(URL inputFile)` - compiles resource referenced through url/uri,
*  `CompilationResult compile(String lessContent)` - compiles a string,
*  `CompilationResult compile(LessSource inputFile)` - extend `LessSource` to add new resource type. 

Each of these method has an additional optional parameter `Configuration options`. Additional options are used only during [source map](https://github.com/SomMeri/less4j/wiki/Source-Maps) generation, so you may ignore them if you do not need it.     
 
Return object `CompilationResult` has three methods: 
* `getCss` - returns compiled css,
* `getSourceMap` - returns [source map](https://github.com/SomMeri/less4j/wiki/Source-Maps),
* `getWarnings` - returns list of compilation warnings or an empty list. 

Each warning is described by message, line, character number and filename of the place that caused it.
  
`Compile` methods differ in one important point: how they handle `@import file.less` statement. In all cases, files referenced by the import statement are assumed to be relative to current file. They are also assumed to have the same type e.g., the first method assumes that all less files imports link files on local filesystem and second method assumes that all less imports reference files by relative urls. The third method is special, it leaves all imports as they are. It does not load and compile any files.         

### Example
````java
// create input file
File inputLessFile = createFile("sampleInput.less", "* { margin: 1 1 1 1; }");

// compile it
LessCompiler compiler = new ThreadUnsafeLessCompiler();
CompilationResult compilationResult = compiler.compile(inputLessFile);

// print results to console
System.out.println(compilationResult.getCss());
for (Problem warning : compilationResult.getWarnings()) {
  System.err.println(format(warning));
}

private static String format(Problem warning) {
  return "WARNING " + warning.getLine() +":" + warning.getCharacter()+ " " + warning.getMessage();
}
````

### Error Handling
`Compile` method may throw `Less4jException`. The exception is checked and can return list of all found compilation errors. In addition, compilation of some syntactically incorrect inputs may still lead to some output or produce a list of warnings. If this is the case, produced css is most likely invalid and the list of warnings incomplete. Even if they are invalid, they still can occasionally help to find errors in the input and the exception provides access to them. 

* `List<Problem> getErrors` - list of all found compilation errors.
* `CompilationResult getPartialResult()` -  css and list of warnings produced despite compilation errors. There is no guarantee on what exactly will be returned. Use with caution.  

## License
Less4j is distributed under following licences, pick whichever you like:
* [The Apache Software License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.txt),
* [Eclipse Public License (EPL)](http://www.eclipse.org/legal/epl-v10.html),
* [Gnu General Public License, Version 3](http://www.gnu.org/licenses/gpl-3.0.html).

## Links:
*  [http://www.w3.org/Style/CSS/specs.en.html]
*  [http://www.w3.org/Style/CSS/Test/CSS3/Selectors/current/]
*  [http://www.w3.org/TR/css3-selectors/] 
*  [http://www.w3.org/wiki/CSS3/Selectors]
*  [http://www.w3.org/TR/CSS2/]
*  [http://www.w3.org/TR/CSS2/syndata.html#numbers]
*  [http://www.w3.org/TR/2012/WD-css3-fonts-20120823/]
*  [http://www.w3.org/TR/CSS21/fonts.html]
*  Comparison of less and sass: [https://gist.github.com/674726]


