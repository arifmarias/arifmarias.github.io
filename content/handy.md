---
title: Handy Functions Reference
---

This page lists some handy functions to use for data wrangling tasks.

## Combining columns

Combining columns can be tricky because merging a blank cell cell with another value results in an error. 
To avoid issues, first facet by blank and combine only non-empty cells with a transform like: `value + " " + cells["col_2"].value`

## De-dupe

On the key column, click "Sort", and choose sort method.
Next to the show rows selection above the table, click on the "Sort" menu. 
Select "Reorder row permanently" (if you do not do this step, sort is just visual and did not transform the data).
On the key column, select "Edit cells" > "Blank down".
Facet on blank, remove all matching rows.

## Compare two columns

Create new "equal" column using expression:
`if(cells["column1"].value == cells["column2"].value, "True", "False")`

## Cross function

Use `cross` to retrieve values from another OpenRefine project based on a common key column. 

1. Open the two projects you want to join
2. Identify the common column to be used as a key. Ensure that it is a unique identifier, or cross won't work how you expect. You can create a new unique column on both projects by adding an index number an existing common column (on common column > Add column based on this column... > `value + "-" + row.index` ).
3. On the project where you want to import values, go to the key column you identified, and Add column based on this column...
4. In the expression window, put: `cell.cross("name_of_other_project", "name_of_key_column").cells["name_of_column_you_want_to_import"].value[0]`

You should have a new column that has the correct values from the other project.

## String + Array functions

A powerful way to interact with large strings (such as the text of poems or web scrape) is to turn them into arrays, then use array functions to manipulate. 
Create an array from any string by using the `split(value, expression)` function. 
The expression is the character or pattern you want to split the string up on, often a new line or a deliminator in a list. 
For example, split on a new line:

`value.split(/\n/)`

Once the cell is an array, it can be rearranged and sliced in many ways with [array functions](https://docs.openrefine.org/manual/grelfunctions/#array-functions).
Then reconstitute the string by using `join()` on the array. 

For example, if we had a list of tags like "dogs; cats; muffins" as a cell value we could put them in alphabetic order using:

`value.split("; ").sort().join("; ")`

Remove the first item in the list:

`value.split("; ").slice(1).join("; ")`

Remove the last item in the list:

`value.split("; ").slice(-1).join("; ")`

Trim the white space for each value:

`forEach(value.split(";"),e,e.trim()).join(";")`

Or filter out values based on condition:

`filter(value.split(";"),v,v.contains("dogs") == false)`

If you had lines of a poem or text as a cell value you could do the same types of operations.
For example, remove the first line of a poem:

`value.split(/\n/).slice(1).join("\n")`

Remove the last two lines:

`value.split(/\n/).slice(-2).join("\n")`

Or trim the white space for each value:

`forEach(value.split(/\n/),e,e.trim()).join("\n")`

## Add leading zeros 

If the column has numbers that should have leading zeros, add the number of zeros it should have in total digits, sliced by value length. 
For example, if you had "12345", "123456", "1234567", and wanted them all to be 8 digits with leading zeros, transform using: 

`"00000000"[0,8-length(value)] + value`

You can also create a new row identifier with leading zeros using the `row.index` variable. 
For example,

`"row_id_" + "0000"[0,4-length(row.index +1)] + (row.index +1)`

## Remove leading or trailing character

In regex `^` is start of string and `$` means end of string, which can be used in a `replace` statement.

Remove "T" from front of string:

`value.replace(/^T/,"")`

Remove trailing period, "." at end of string:
 
`value.replace(/\.$/,"")`

(note the "." needs to be escaped with `\` since it has a meaning in regex)

## Facet by facet count

Sometimes you have a column with many repeating values, that you might explore using a text facet. 
In the text facet pane you can sort by facet count, but you would have to manually select each if you wanted a subset based on the facet count.
To select a group of rows based on the facet count of a values in a column: 

First, if you just need all the values with > 1 count, you can use the built in Facet > Customized facets > Duplicates facet. 
This returns "true" for rows with > 1 count, false if the value is unique.

Second, if you need a subset based on the count, create a new column using the `facetCount` function.
On the column you want a count for, Edit column > Add column based on this column, and use:

`value.facetCount("value","name_of_the_column")`

The result will be a number (same as the "count" given in facet pane), which you can then filter with a numeric facet.
(note in this context facetCount seems a bit non-intuitive since you have provide "value" and the name of the column again--facetCount is set up with flexibility to do some more complicated operations by adding an expression to the value or matching values in a different column)

## Parse JSON

It is common to get JSON data when fetching from APIs using Refine. It's easy to grab specific dictionary values out of JSON cells using the built in JSON parse function. From the column with JSON, create a new column and transform with `value.parseJson().get('key')`, where 'key' is the dictionary key you want to extract. 

For example, if the cell contained
`{ "type" : "dog", "color" : "brown", "size" : "large" }`, 
and your transform was`value.parseJson().get('color')`, 
you would get the value "brown" in your new column. (*note*: if your key does not have spaces, you can use the shorter version like `value.parseJson().color`)

To get multiple values from the same key, combine with `forEach()`.
For example, to extract all the keywords from a cell with the JSON
`{'language': 'en', 'keywords': [{'text': 'dogs', 'relevance': 0.979292}, {'text': 'muffins', 'relevance': 0.977987}, {'text': 'cats', 'relevance': 0.969001}, {'text': 'idaho', 'relevance': 0.967973}] }`,
transform with `forEach(value.parseJson().keywords,v,v.text).join("; ")`, resulting in the new cell value of `dogs; muffins; cats; idaho`.

## Common HTML parsing

Combining "Create new column by fetching from URL" and the `parseHtml()` GREL function is a powerful and flexible method to harvest data from the web or scrape sites. 
Always remember to use `.toString()` or `.join("|")` at the end of your parsing statements or you will end up with empty cells even through your html parsing is correct!

I often use these GREL statements to extract stuff out of HTML:

- get all image src out: `forEach(value.parseHtml().select('img'),i,i.htmlAttr('src')).join("; ")`
- get all links out: `forEach(value.parseHtml().select('a'),i,i.htmlAttr('href')).join("; ")`
- cells out of a table rows: `forEach(value.parseHtml().select('tr'),i,i.select('td')).join("; ")`
