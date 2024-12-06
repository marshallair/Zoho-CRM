# Zoho-CRM
Various functions used within Zoho CRM projects

FieldMerge()

	FieldMerge() provides a "merging" language using strings and field data, along with functions and operators, to provide (hopefully) a robust ability to aggregate
	Zoho data using Deluge into chunks of string data. These would typically be stored in multi-line fields within the CRM environment, but the function outputs a String
	that can be used anywhere strings are use.

	The FieldMerge() arguments are:
		String MergeText												A string representing the "merge language" of FieldMerge() to be parsed for output
		String Data															JSON or Map data passed to FieldMerge to provide the data to replace field blocks in MergeText with actual field values
		String OpenDelim (optional)							A open block delimiter. If left null, "{" is the default
		String CloseDelim (optional)						A close block delimiter. If left null, "}" is the default
		String FunctionDelim (optional)					A delimiter used to identify function name beginnings and ends. If left null, "$" is the default.

	The FieldMerge "language" is based on the concept of blocks, functions, operators, and string text. Hereafter, the default OpenDelim, CloseDelim, and FunctionDelim
	are used in the descriptions.

FieldMerge() BLOCKS

	A block, specified hereafter as Block{}, is a chuck of text that is treated as identifiably separate from the surrounding MergeText information.
	Blocks can contain anything. Examples:
		String text:																{Hello world!}
		Functions to retrieve field data:						${firstname} or $FIELD${firstname}
		Arguments for functions:										$IIF$({${Dept}==Sales},${Revenue},{})
		Other, nested blocks:												Level0Text{Level1Text{Level2Text}{MoreLevel2Text}{{Level3Text}}}
		
	Between the ability to nest blocks, and use functions, any output can be achieved, including JSON or HTML.

FieldMerge() FUNCTIONS

	The Functions used within the FieldMerge() language always start with $ and end with $, as in $FUNCTIONNAME${}.
	Parameters for functions appear as ordered blocks within the function, such as $IIF${{condition}{eval_if_true}{eval_if_false}}
	Some functions can have no arguments, or unlimited arguments, such as $AND${{arg1}{arg2}...{argN}}.
	Other functions can have optional arguments, such as $F${{fieldname},{optional_eval_if_null}}.
	A few functions have shortened versions. For example, ${} is the shortcut for the $F${} function.

	The functions are:
		NAME							DESC															ARGS																					RETURNS
		-----							-------------											-----------------------												---------------
		$IIF$ or $IF$			Immediate IF function							{condition}{eval_if_true}{eval_if_not_true}		{eval_if_true} for {condition}=true, all others {eval_if_not_true}				
		$AND$							logical AND												NONE or unlimited															TRUE as default until a false is found
		$NAND$						logical not AND										NONE or unlimited															FALSE as default until a false is found
		$OR$							logical OR												NONE or unlimited															FALSE as default until a true is found
		$NOR$							logical not OR										NONE or unlimited															TRUE as default until a true is found
		$XOR$							logical exclusive OR							{boolean1}{boolean2}													TRUE for different, FALSE for same
		$NOT$							logical not												unary function so {boolean}										Opposite of {boolean}
		$F$ or $					field value												string with function name											returns value of field or null if an error
																												-OR- {functionname}{eval_if_blank_or_error}		returns value of field, or second argument if blank or an error

		New functions can be easily written by including a new name in the if else block for parsing after an Open Delimiter is found.

FieldMerge() OPERATORS
	Between blocks{}, operators can be used to create compound evaluations:
		OPERATOR						DESC															RETURNS
		+										add or concatenate								if one value is a string, concat and return string, otherwise, add and return number
		-										subtract													number or null if both values are not numbers
		*										multiply													number or null if both values are not numbers
		**									raise previous to power						number or null if both values are not numbers
		/										divide														number or null if both values are not numbers
		--									unary negative										negative of number or number or null value is not a number
		&&									logical AND												boolean (defaults as true unless one argument is error, null, or false)
		||									logical OR												boolean (defaults as false unless one argument is boolean true)
		!										logical NOT												boolean (defaults as false unless the block is boolean false)

FieldMerge() OPERATION
	Because Deluge does not support recursion, and FieldMerge is intended to support unlimited nesting, a List is used like a stack to build the nested evaluations
	into a structure, then remove them from the list when evaluated into items in which they are nested. FieldMerge processes the entire MergeText string using
	looped searches for the OpenDelim and CloseDelim characters (hereafter we'll just used { and } to refer to these, although they can be changed).

	The concept is simple: if "{" is found, then an element is "poked" to the list, and the MergeText immediately preceding the { is parsed to determine if the block{}
	is a function, or is preceded by an operator. If a function is identified, then an additional list element with a text argument identifyer is pushed (for the next
	expected argument), and a boolean is set so the next pass knows the next block{} found is for this argument.

	Operators are treated like functions, in that they can be processed in the above manner. However, for binary operators, the previous stack value is going to be used,
	and therefore the current list item evaluation must also use the previous list item in the evaluation. For unary operators, they are simply treated like functions
	during evaluation.

	If a function is not identified, the block is processed as a container of text or other blocks in heirarchy.

	Once a } is found, the current block operation is completed, the current list value is changed to what is evalued based on the current identifyer in the stack,
	and then "popped" depending on whether it is required for that particular identifyer. 

	If an evaluated value is left in the stack, then this becomes something that can be passed back "up the stack" on subsequent passes. This allows for multiple
	list items to be considered ordered arguments for a function several list items earlier. Upon the last argument being evaluated, the ending function code
	will pass back and "pop" all the values until the function has been completely evaluated, which, in turn, then replaces the function identifyer in the stack
	with its return value, and the process continues.

	This appraoch allows nesting of blocks{} but also blocks{} within argument blocks withing blocks{}, ad infinitum. It just evaluates the current block as
	a container (process now) or an argument (evaluate now and store in list for later).

	When StackLevel is at 0, we know we are at base level and whatever comes from that evaluation is concatenated (appended) to the existing output.

	When the entire MergeText has been evaluated, the final output string is returned.

	In theory, an unlimited number of functions and operators can be defined, nested in unlimited fashion, and processed WITHOUT requiring recursion.

FieldMerge() serves a similar purpose to ConcatFields, in that it mixes field data with string literals to create an output. Eventually, I'll get rid of ConcatFields() altogether.

----------------------------------------

ConcatFields()
ConcatFields() is standalone function that retrieves a list of fields and concatenates them
 with while using a formatting string (labels and line breaks, typically, but any can be used). A simplified version that just uses field names
 for labels with a colon and space appended, the field values are next, followed by the default line break text. This is the default when
 no formatting string in sFieldList is provided (such as "Fiend+Name1,,Fieldname2,,Fieldname3,,...").
 
 Blank values can be surpressed or allowed. If suppressed, the entire section associated with that field is omitted.
 
 ConcatFields() is a useful function to  aggregate several fields into a single text field that can be displayed, edited, and
 output for reports or merges. It is also useful for aggregating multiple-line addresses into single text blocks.
	
	Format:
    <stringvar>=
	ConcatFields(
		sModule as string,							the name of the module to retrieve
		ID as int,									the ID of the record to be retrieved from sModule
		sFieldList as string,						see below
		sDelim as string="{,}",               		the delimiter when parsing sFieldList
		sLine as string="<br>",						the line break insert text
		sFinsert as String="{$}",					the formatting string to insert field data
		sNinsert as String="{_}"					the formatting string to insert field name data
		sLinsert as String="{nl}"					the formatting string to insert a line break
		bIgnoreBlanks as bool=TRUE					ignore formatting for a field that is blank = "" or null
		sBlanks as String=""						if blanks are not to be ignored, what should be put in their place
		

sFieldList is in itself a string parameter list, as follows:
	The sFieldList parameter assumes substrings as parameters in pairs with "{,}" as the default delimiter (set by sDelim),
	namely: FieldName1{,}Format2{,}FieldName2{,}Format2{,}...FieldName_n{,}Format_n
	(note: fieldnames that are Date/Time fields must be preceded by "{dt}" - Example of sFieldList with a date/time field and a format string = "{dt}StartDateTime{,}Start time: {f$}MMM/dd/yyyy hh:ss{\f$}")

	The sFieldList internal delimiter is set by sDelim, which defaults to "{,}" to avoid the likelihood that it will be used in a parameter
    passed accidentally, but a simple comma will suffice if you are guaranteed not to be using a comma in any portion of sFieldList
	besides the delimiters.

    In essence, the sFieldList format entries are treated like a literal string with inserts for
		Field Value (usually "{$}" but set by sFInsert).
		Field Label Name with underscores replaced by spaces (usually "{_}" but set by sNinsert).
		Line breaks (usually "{nl}" but set by sLinsert).
			Linebreaks could also be used for a third generic insert that doesn't break lines.
			The default line break is set by sLine, which itself defaults to "<br>",
			 which is the RichText line break, but we could use "\n" for simple text multiline fields, or any other string.
			The main difference between field and name inserts vs line inserts is that line inserts are always the same
			 value, set by sLine.
		FormattedToString inserts are treated differently. There is no way to change the default insert code with the parameters. The Formatted toString
		 inserts have the format {f$}format_string{\f$} where the format string will be used on the field to do a .toString() conversion.

	SIMPLE VERSION: If the format portion is omitted, the format defaults to sNinsert: sFinsert sLinsert, or with defaults "{_}: {$}{nl}".
	 This allows a list of fields and blank values to quickly create a vertical list of fields.
	An example of sFieldList values and their outputs, assuming it's run on multiple records:
		SFieldList "Name{,}{_}: {$}{nl}{,}Date_of_Birth{,}Born on the date of {$}{nl}{nl}"
		 when used for two normal records and a third record without a name (sIgnoreBlanks=FALSE and sBlank="FIELD EMPTY") as in report might look like:
			
			Name: Marshall Henley
			Born on the date of 3/18/1970

			Name: Amy Henley
			Born on the date of 2/21/1972

			FIELD EMPTY
			Born on the date of 12/25/2000

			---
			... with the actual string returned being:
			Name: Marshall Henley<br>Born on the date of 3/18/1970<br><br>Name: Amy Henley<br>Born on the date of 2/21/1972<br><br>FIELD EMPTY<br>Born on the date of 12/25/2000

	The same example with sIgnoreBlanks=True (default)would look like:
			
			Name: Marshall Henley
			Born on the date of 3/18/1970

			Name: Amy Henley
			Born on the date of 2/21/1972

			Born on the date of 12/25/2000

			---
			... with the actual string returned being:
			Name: Marshall Henley<br>Born on the date of 3/18/1970<br><br>Name: Amy Henley<br>Born on the date of 2/21/1972<br><br>Born on the date of 12/25/2000

	The power of these formatting strings is they can be stored in text fields in subForms,
	for example, and used to aggregate subForm data with powerful formatting, including Rich Text formats for output to emails, etc.
	
	NOTE: Because of a known problem with the way Deluge returns date/time values which won't work with Deluge date and time functions, or with toString(), any date/time fields need to use a special field name to insert them, as indicated above.
	The field data returned from a DateTime record doesn't match the format needed by Deluge to properly process using DateTime functions, so it must first be changed using the .toTime() function,
	then can be manipulated and reformatted back to the record-based standard. For example:
		updated_date_time = record.get("Date_Time_Field").toTime("yyyy-MM-dd'T'HH:mm:ss").addMinutes(90).toString("yyyy-MM-dd'T'HH:mm:ss+10:00")
*/

