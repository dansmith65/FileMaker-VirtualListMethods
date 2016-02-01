# FileMaker-VirtualListMethods

An overview of the different methods that can be used to implement the virtual list technique.


## Status of this Document

rough draft; is likely missing information


## Global or Local Variables?

Global variables are typicaly used for virtual lists, but there are some situations where local variables will work just as well, or even better.

### Pros of local variables:

1. Don't need to be deleted; they are cleared when the script exits.
    - This has more of an impact when discussing repeating variables, since there are more values to clear vs return-delimited values in a single variable, where there are fewer variables to clear.

### Cons of local variables:

1. Are only visible while a script is running.
    - Unless they are scoped to something other than a script, but I'm not going to go down that train of thought.
    - This means the same script that collects the data must be running while the data is viewed. This may not matter for a print layout, but obviously would not work very well for something like Master Detail.


## Return-delimited or Repeating Variables?

Return-delimited seems to be the most common method, but repeating variables is also an option.

### Pros of return-delimited:

1. Many common methods of retrieving the data result in a return-delimited list: 
    - List( ) of related records
    - List of Summary field (to get the found set)
    - ExecuteSQL

### Cons of return-delimited:

1. Empty values can cause rows to be miss-alligned.
2. Variables are immutable, so appending data to a variable get's slower as the data get's larger. In other words: it can be slow with large lists when creating the list via looping through records.
3. Value cannot contain a return (this issue can be bypassed by substituting/encoding returns, both of which have their own issues).

### Pros of Repeating:

1. Never have to worry about empty values.
2. Fast to create, IF you have to loop through records.

### Cons of Repeating:

1. Many different values to clear since each value is in a separate variable.


## Serialized Text

\# functions, json, etc. This may be useful if your source material is already encoded (e.g., receiving a JSON array).


## ID's or Text Values?

If collecting id's, need a relationship between the virtual list table and the related table to display the data. If collecting text values, no relationship is necessary.

TODO: debate pros vs cons


## Single variable or Multiple?

I typically use a separate variable for each field in the VirtualList table. Another option is to encode all data in a single variable.

TODO: debate pros vs cons


## Multiple VirtualList Fields

I typically create multiple, generic fields in the VirtualList table like so:

- id1
- id2
- text1
- text2
- date1
- etc.

TODO:  how many fields is too many?

## Method for extracting array values to repeating field

When the result is just required for display purposes, I use a virtual array where values in each row are separated by the pipe character. This formula auto-extracts each value into repetitions of a field. If you need more values to display then just add another repetition.

```
// vListWhole is field storing individual repetition with sub-values separated by "|"

If ( 
	Let ( [ 
		_count = PatternCount ( Extend ( vListWhole ) ; "|" ) ;
		_rep = Get ( CalculationRepetitionNumber ) ;
		_start = Max( 0 ; Position ( Extend ( vListWhole ) ; "|" ; 1 ; _rep - 1 ) ) +1 ;
		_end = Position ( Extend ( vListWhole ) & "|" ; "|" ; 1 ; _rep ) ; 
		$extract = Middle ( Extend ( vListWhole ) & "|" ; _start ; _end - _start )
	] ;
	
	_count > 0 
	
	) //end let
	
;
	$extract
	
; //else

	If ( Get ( CalculationRepetitionNumber ) = 1 ;
		vListWhole 

	; //else
		"" 
	) //end if
	
) //end if
```


## Primary Key vs Get( RecordID )

TODO


## VirtualList::index field vs Get( RecordNumber )

I've never actually used `Get( RecordNumber )` instead of an index field. [VIRTUAL VALUE LIST](http://www.modularfilemaker.org/module/virtual-value-list/) uses it, though.

TODO

## Notes

1. You can't perform a find in the VirtualList table in a file opened on a remote server. This is because the data only exists locally but a find in an unstored calculated field is processed on the server.
