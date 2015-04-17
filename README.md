# srcML-XSLT-Tutorial
How to use XSLT with srcML

XSLT is a language for transforming XML documents. Transformations usually concern converting XML to HTML or other XML documents. In this tutorial, we’ll be focusing very heavily on XML to XML transformations and particularly we’ll be focusing on transformations applied to srcML archives.

XSLT makes heavy use of XPath in order to do its matching. In fact, XSLT follows the typical transformation idiom directly; match and transform. Match with xpath, transform with xslt.

For this tutorial, we’ll be implementing a number of small transformations and detailing what’s going on in each. In this way, we’ll learn (by example and explanation) how to use XSLT.

First things first. In order to declare a document to be an XSL stylesheet, we need a specific tag at the top:

```xslt
<xsl:transform version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
```

This gives us access to XSLT elements, attributes and features we’ll need to apply it.

Now there’re several tags we’ll learn about and we’ll summarize them here in order of which we’ll learn.
```xslt
<xsl:template> -- Defines a set of rules to apply when a specific node(set of nodes) are matched.
<xsl:value-of> -- Extracts the value (Text) of the selected node
<xsl:for-each> -- Iterative structure to search set of nodes
<xsl:if> -- template surrounded by ‘if’ only applies if condition is true
<xsl:choose> -- used with <when> and <otherwise> to express multiple conditions
<xsl:copy> -- Creates a copy of the current node (without attrs and children)
<xsl:copy-of> -- Creates copy of current node (with children and attrs)
<xsl:text> -- Writes literal text to output.
<xsl:function>
```
Now, to start, we’re going to do a very simple XSLT script to transform expressions of the form:

```c++
int number = rand();
```
into the form:

```c++
int number;
number = rand();
```

The first thing we need to do is copy the entire document (since we need a copy in-memory to work on). To do this we apply what’s called an identity copy:

```xslt
<!-- default identity copy -->
<xsl:template match="@* | node()">
   <xsl:copy>
       <xsl:apply-templates select="@* | node()"/>
   </xsl:copy>
</xsl:template>
```

Which is a pretty standard operation. We’ll find that, as mentioned before, xslt follows the ‘match and transform’ idiom. So it makes sense that the first thing we need to do is match the nodes we want to apply a transformation to:

```xslt
<xsl:template match="src:decl_stmt">
</xsl:template>
```

So using <xsl:template> we specify that we want to match decl_stmts within the srcML archive. After we’ve matched the context, any xpath we write within the <template> will now be built on top of this context (so we don’t need to rewrite it). Let’s get the name of the type next:

```xslt
<xsl:copy-of select="src:decl/src:type/src:name"/>
```

And now add a space between the type and the name of the variable

```xslt
<xsl:text> </xsl:text>
```

Next we add the name of the variable

```xslt
<xsl:copy-of select="src:decl/src:name"/>;
```

This finishes the first part of our transformation. Now the delcaration:

```c++
int number = rand();
```

Would be transformed into:

```c++
int number;
```

As a side note, this would all be copied without proper spacing going in front of the declaration… to copy the spaces properly, we require the xslt tokenize function. All in all, our code so far looks like this:

```xslt
<xsl:template match="src:decl_stmt[src:decl/src:init]">
<!-- Copy the declaration, without any part of the initialization -->
<xsl:text>
</xsl:text>
    <xsl:value-of select="str:tokenize(preceding-sibling::text(), $newline)[2]"/>
    <xsl:copy-of select="src:decl/src:type/src:name"/>
    <xsl:text> </xsl:text>
    <xsl:copy-of select="src:decl/src:name"/>;
```

Now we’re ready to copy the initialization:

```xslt
<xsl:variable name="ndecl">
        <xsl:value-of select="src:decl/src:name[1]"/>
        <xsl:value-of select="src:decl/src:init"/>;
</xsl:variable>
```

We also show off the use of xslt variables here. Now the variable contains the text found in both of these xpath expressions. Again, we need tokenize so that the line:

```xslt
number = rand()
```

is properly indented. Then we output the value of the variable “ndecl” and close the template

```xslt
<!-- Copy the generated try catch with the indentation of the original statement -->
    <xsl:value-of select="str:tokenize(preceding-sibling::text(), $newline)[2]"/>
    <xsl:value-of select="src:indent(src:indentation(.), $ndecl)"/>
 </xsl:template>
```

This completes our first transformation.

All in all, our code looks like this:

```xslt
<xsl:stylesheet
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
	xmlns="http://www.sdml.info/srcML/src"
	xmlns:src="http://www.sdml.info/srcML/src"
	xmlns:cpp="http://www.sdml.info/srcML/cpp"
	xmlns:lit="http://www.sdml.info/srcML/literal"
	xmlns:op="http://www.sdml.info/srcML/operator"
	xmlns:type="http://www.sdml.info/srcML/modifier"
	xmlns:func="http://exslt.org/functions"
	xmlns:common="http://exslt.org/common"
	xmlns:str="http://exslt.org/strings"
	xmlns:set="http://exslt.org/sets"
        extension-element-prefixes="func"
	version="1.0">

	<xsl:include href="srcmltrans.xsl"/>
	<!-- default identity copy -->
	<xsl:template match="@*|node()">
		<xsl:copy>
	  	<xsl:apply-templates select="@*|node()"/>
		</xsl:copy>
	</xsl:template>

	<!--
    	Match declarations with a new operator in the declaration.
	-->

<xsl:template match="src:decl_stmt[src:decl/src:init]">
<!-- Copy the declaration, without any part of the initialization -->
<xsl:text> 
</xsl:text>
	<xsl:value-of select="str:tokenize(preceding-sibling::text(), $newline)[2]"/>
	<xsl:copy-of select="src:decl/src:type/src:name"/>
	<xsl:text> </xsl:text>
	<xsl:copy-of select="src:decl/src:name"/>;
<!-- Wrap a try catch around the initialization of the variable, now in separate statements -->
	<xsl:variable name="ndecl">
		<xsl:value-of select="src:decl/src:name[1]"/>
		<xsl:value-of select="src:decl/src:init"/>;
	</xsl:variable>
<!-- Copy the generated try catch with the indentation of the original statement -->
	<xsl:value-of select="str:tokenize(preceding-sibling::text(), $newline)[2]"/>
	<xsl:value-of select="src:indent(src:indentation(.), $ndecl)"/>
 </xsl:template>

</xsl:stylesheet>
```


