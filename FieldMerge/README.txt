# Zoho-CRM
FieldMerge()
    FieldMerge() provides a "merging" language using strings and field data, along with functions and operators, to provide (hopefully) a robust ability to aggregate
    Zoho data using Deluge into chunks of string data. These would typically be stored in multi-line fields within the CRM environment, but the function outputs a String
    that can be used anywhere strings are use.

    FieldMerge() returns a string that represents the full processing of MergeText against the Data.

    The FieldMerge() arguments are:
        String MergeText										A string representing the "merge language" of FieldMerge() to be parsed for output
        String Data												JSON or Map data passed to FieldMerge to provide the data to replace field blocks in MergeText with actual field values
        String OpenDelimiter (optional)							A open block delimiter. If left null, "{" is the default
        String CloseDelimiter (optional)						A close block delimiter. If left null, "}" is the default
        String FunctionCharacter (optional)						A delimiter used to identify function name beginnings and ends. If left null, "$" is the default.

    The FieldMerge "language" is based on the concept of blocks, functions, operators.

FieldMerge() BLOCKS

    A block is a chuck of text that is treated as identifiably separate from the surrounding MergeText information.
    Blocks can contain anything. Examples:
        String text:                          {Hello world!} => "Hello World!"
        Functions to retrieve field data:     ${firstname} or $F${firstname}te => "Jerry"
        Function Arguments and operators:     $IF${{${Dept}==Sales}${{Revenue}{{$N}: {$}}}{Not in sales department.}} => "Sales: $25,000"  -OR-  =>"Not in sales department."
        Embedded or sequential blocks:        Level0Text {Level1Text {Level2SequentialText }{MoreLevel2SequentialText }{{Level3Text}}}. => "Level0Text Level1Text Level2SequentialText MoreLevel2SequentialText Level3Text."
        
    Between the ability to nest blocks, and use functions, any output can be achieved, including JSON, HTML, or Rich Text.

FieldMerge() FUNCTIONS

    Details of functions and their parameters are found in addendum A.

    The Functions used within the FieldMerge() language generally start with $ and end with $, as in $FUNCTIONNAME${}.
    There is one exception, {{}} for simple fields.
    Parameters for functions appear as sequential blocks within the function, such as $IF${{condition}{eval_if_true}{eval_if_false}}
    Some functions can have no arguments, or unlimited arguments, such as $AND${{arg1}{arg2}...{argN}}.
    Other functions can have optional arguments, such as $F${{fieldname},{optional_eval_if_null}{optiona_eval_if_error}}.
    Some functions contain placeholders for use within parameters, such as {$} for field data or {$N} for the API field name.
    A few functions have shortened versions. For example, ${} is the shortcut for the $F${} function.

    The functions are:
        NAME                     DESC                                ARGS                                                                RETURNS
        -----                    -------------                       -----------------------                                             ---------------
        $IF$                     Immediate IF function               {condition}{eval_if_true}{eval_if_not_true}                         {eval_if_true} for {condition}=true, all others {eval_if_not_true}				
        $AND$                    logical AND                         NONE or unlimited                                                   TRUE as default until a false is found
        $NAND$                   logical not AND                     NONE or unlimited                                                   FALSE as default until a false is found
        $OR$                     logical OR                          NONE or unlimited                                                   FALSE as default until a true is found
        $NOR$                    logical not OR                      NONE or unlimited                                                   TRUE as default until a true is found
        $XOR$                    logical exclusive OR                {boolean1}{boolean2}                                                TRUE for different, FALSE for same
        $NOT$                    logical not                         unary function so {boolean}                                         Boolean of {boolean}
        {{modulename.fieldname}} simple field value                  field API name                                                      Returns the value of Fieldname or a blank if null or an error.
        $F$ or $                 field value                         {field API name}{not blank},{blank}{error}
                                                                    -OR- {functionname}{eval_if_blank_or_error}                          Value of field if only field name is present (or blank if error or null).
                                                                                                                                         The module is assumed to be the default module, unless specified. The format is moduleAPIname.fieldAPIname.
                                                                                                                                         If not null block is present, then it is returned.
                                                                                                                                         {$} is placeholder for field value.
                                                                                                                                         {$N} is placeholder for field API name without module name.
                                                                                                                                         If null block is present, then it is evaluated and returned for a null field value, including {$N}
                                                                                                                                         If error block is present, then it is returned if there is an error, such as field name doesn't exist,
                                                                                                                                             {$N} can be used and {$E} is the placeholder for the error value.
        $DEFVAR$                 defines a variable                  variable name                                                       nothing or error value if not a valid variable name. Can contain alpha, numeric, and "_" characters.ANDAND
        $GETVAR$                 returns a variable value            variable name                                                       returns a variable value if only variable name is present (or blank if error or null)
                                                                                                                                         If not null block is present, then it is returned. {$} is placeholder for value, {$N} is placeholder for variable name.
                                                                                                                                         If null block is present, then it is returned for a null variable value, {$N} is placeholder for variable name.
                                                                                                                                         If error block is present, then it is returned if variable name doesn't exist, {$N} is placeholder for variable name.


FieldMerge() OPERATORS
    Between blocks{}, operators can be used to create compound evaluations:
        OPERATOR	DESC											RETURNS
        +			add or concatenate								if one value is a string, concat and return string, otherwise, add and return number. If both are boolean, perform logical AND.
        -			subtract										number or null if both values are not numbers.
        *			multiply										number or null if both values are not numbers.
        **			raise previous to power							number or null if both values are not numbers.
        /			divide											number or null if both values are not numbers.
        --			unary negative									negative of number or number or null value is not a number. If boolean, logical NOT.
        &&			logical AND										boolean (defaults as true unless one argument is error, null, or false)
        ||			logical OR										boolean (defaults as false unless one argument is boolean true)
        !			unary logical NOT								boolean (defaults as false unless the block is boolean false)
        ==          logical equals                                  true if two values are equal, false otherwise. Errors return a false.

NEW FieldMerge() FUNCTIONS & OPERATORS
    New FieldMerge functions and operatorsw can be easily written inside the PreprocessBlock() and PostProcessBlock() functions. Functions and operators can create any parameter
    structure required in the ResultsMap, and process them as needed using the BLOCKNAME attribute for postprocessing.

FieldMerge() OPERATION
    FieldMerge() uses "blocks", which can contain anything, {anything}. Blocks can be containers for other blocks. Blocks can be functions which contain other block as parameters.
    Blocks that are parameters can contain other blocks, and so on. A block inside another block is called an embedded block. A block folowing another block is called as sequential block.
    By definition, a list of parameters for a function block would require that block to process a series of sequential blocks as parameters.

    FieldMerge() can also discern "chunks" which are pieces of text not formally within a block. They are considered static blocks of text. Some are ignored if found in certain functions.
    An example distinguishing blocks from chunks would be: {This is a block, }but this is a chunk.{ This is another block}, while this is another chunk.

    THE ALGORITHM
    Because Deluge in CRM does not support recursion, and unlimited string lengths as parameters are allowed, and FieldMerge is intended to support unlimited block nesting,
    a List (considered an Index) is used as an Key value in a Map (ResultsMap) to build the nested evaluations into a structure, then remove them from the structure when evaluated.
    BLOCKTYPE values are placed in the new ResultsMap entries until the block is complete, then processed based on the BLOCKTYPE found, as well as function parameters and their defaults.

    Chunks, when found, are either ignored (with some functions) or treated as plain sequential blocks of static text.

    ResultsMap, a map with the indexed block key-value map, as well as key-values for all pertinant processing is used. ResultsMap also carries the data. It is established in the FieldMerge()
    function and then passed to subsequent standalone function calls, and then passed back as the result.

    The root index in the block map for ResultsMap is [0] (as opposed to what it contains, such as {[0,1],[0,2]...[0,n]}). When processing is finished, the only key-value left should be
    ResultsMap.get([0]) and therefore this is what is used to return the value of FieldMap() upon completion, barring errors being passed back.

    Because escaping of the block delimiters and field characters are allowed, there is a function to handle this, too, EscapeCharacters().

    Within FieldMerge(), there is a container function, ProcessBlock(), which can be modified to call "fmfunc_" functions (see below) to implement new
    functions or arguments and expand the abilities of FieldMerge. ProcessBlock() calls these fmfunc_ functions (see below) to create
    the map entries when the first parameter of ProcessBlock is true (herafter called PRE), and then process the map values into a final result during
    ProcessBlock(false,...) ("POST" hereafter). To simplify the mapping and processing of key-value pairs, both ProcessBlock and fmfunc_ functions during
    PRE and POST, fmfunc_ functions actually effectively two functions, one for PRE, and one for POST. These were kept in one function to allow quick edits
    to both the pre and post versions as function parameters are added, etc. To clarify...

    This appraoch allows nesting of blocks{} but also blocks{} within argument blocks withing blocks{}, ad infinitum. It just evaluates the current block as
    a container (process now) or an argument (evaluate now and store in list for later).

    When StackLevel is at 0, we know we are at base level and whatever comes from that evaluation is concatenated (appended) to the existing output.

    In theory, an unlimited number of functions and operators can be defined, nested in unlimited fashion, and processed without recursion.

FUNCTIONAL RESPONSIBILITIES
FieldMerge()
    FieldMerge processes the entire MergeText string using looped searches for the OpenDelimiter and CloseDelimiter characters (hereafter we'll just used { and } to refer to these,
    although they can be changed as parameters in FieldMerge). If "{" or "}" is found, then a block is begun or ended and is processed using ProcessBlock(). Blocks need to be discerned
    as being sequential or embedded, so the LastDelimiterWasOpen boolean is used to make that determination. A { following a { is embedded. A { following a } is sequential.
    A } following a } is the end of an embedded. A } following a { is the end of a sequential. FieldMerge() is concerned with:
    - Looping to find the beginning or end of blocks
    - Discerning what type of block is being begun or ended.
    - Loading ResultsMap with the necessary information each loop to be processed, including the type of processing to be done
        (start or end a sequential  or embedded block, established by an integer 1 through 4 stored in BlockProcess).

ProcessBlock()
    ProcessBlock() is concerned with:
    - Defining the "chunk" (the text, if any, between the last delimiter and the one just found), placing it in ResultsMap,
    - Discerning which processing is to be done by passing it off to the correct fmfunc_ function for pre or post processing.
    - Resetting the cursor when done.

        ProcessBlock() with IsPRE==true
        ProcessBlock() looks to see if it's a function block, or is preceded by an operator, and then proceeds to appendi to the map with BLOCKTYPES and/or default values in the map, ResultsMap with list Index as key.
        FieldMerge() "functions and operators" are processed at this time. These can be added by creating the functions as standalones that are accessed by PreprocessBlock() and PostprocessBlock().
        The List used as key values for the ResultsMap, and the MergeText immediately preceding the { is parsed to determine if the block{}
        is a function, or is preceded by an operator.

        Operators are treated like functions, in that they can be processed in the above manner. However, for binary operators, the previous ResultsMap sequential block value is going to be used,
        and therefore the current list item evaluation must also use the previous list item in the evaluation. For unary operators, they are simply treated like functions
        during evaluation. All the standalone procesessing functions called by PreprocessBlock() should have a prefix of "fmfunc_", for clarity.

        If a function is not identified, the block is processed as a container of text or other blocks in heirarchy.

        ProcessBlock() with IsPRE==false
        Once a } is found, the current block operation is completed by Postprocess block, looking at the value in the block and the text in MergeText. The current ResultsMap values are then replaced
        by the result of the functions called by PostprocessBlock().

        If a "}" is found after a preceding "}", then we know the block being concluded contains one or more embedded blocks. Each block in the sequence is evaluated based on BLOCKTYPE
        and the results are placed in the finished block value, with the embedded block values being deleted. Sometimes, block values are ignored, such as when an IF condition is true or false,
        and PostprocessBlock will send a delete Index list back to FieldMerge() to stop unnecessary processing of blocks.
        PostprocessBlock()

        If an evaluated value is left in the ResultsMap, then this becomes something that can be passed back "up the stack" on subsequent passes. This allows for multiple
        list items to be considered ordered arguments for a function several list items earlier. Upon the last argument being evaluated, the ending function code
        will pass back and "pop" all the values until the function has been completely evaluated, which, in turn, then replaces the function identifyer in the stack
        with its return value, and the process continues.

fmfnc_ FUNCTIONS
    fmfunc_ functions are concerned with:
    - If IsPRE is true, loading ResultsMap with key-maps to be processed later in post processing, checking first to see if
        they already exist (in the case of an argument, to a function, for example).
    - If IsPRE is false, processing the block just completed with the information as defined in it's block text, and collapsing/
        processing any remaining embedded blocks within the current block. Once an end of block is processed, there should never be
        any remaining embedded blocks entries (children of that block) in ResultsMap.

OTHER STANDALONE FUNCTIONS
To aid in discerning certain processing conditions, there are block counting functions, BlockCount(), SequentialBlockCount(), and BlockDepth().

----------------------------------------

ConcatFields()

    FieldMerge() serves a similar purpose to ConcatFields, in that it mixes field data with string literals to create an output. Eventually, we will get rid of ConcatFields() altogether.

ConcatFields() is standalone function that retrieves a list of fields and concatenates them
 with while using a formatting string (labels and line breaks, typically, but any can be used). A simplified version can be used that just outputs field names
 for labels with a colon and space appended is, field values next, followed by the default line break text. This is the default when
 no formatting string in sFieldList is provided (such as "Fieldname1,,Fieldname2,,Fieldname3,,...").
 
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
    namely: FieldName1{,}Format1{,}FieldName2{,}Format2{,}...FieldName_n{,}Format_n
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

            Name: Joe Henley
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

ADDENDUM A: FieldMerge() Functions/Operators descriptions and arguments

DEFAULT
FIELD

ADDENDUM B: Noted problems with Deluge processing:
Lists used as Map keys:
    FieldMerge consistently uses an Index list as a map key to allow levels to be stored within the Indices, not just sequential location.
    Deluge maps with an a list used as key value(s) apparently use an internal pointer or something, because if we change the Index variable used
    to create the map key-value pair with .add(), the next time we inspect the newly added variable the key has changed to whatever value the Index list
    variable has changed to.
    
    Therefore, we have to "clone" the list to use it using duplicate().
    
    Also, without using toList() with this function, occasionally the new index
    elements are converted to strings (another bug we assume as it happened arbitrarily), such as ["0","1"] instead of [0,1], for example. This seemed arbitrary.
    
    Therefore, we always use...
        listvar.duplicate(0).toList() 
    ...when assigning a list to a map key.

Problems with Deluge IfNull():
    In some cases, ifnull was returning the first parameter even though isnull() on the same parameter was returning true. This seemed arbitrary, and was not consistent.
    Therefore, instead of...
        varname=ifnull(varname,"default value")
    ... we consistently use (particularly for standalone function parameters):
        if(varname=="")
        {
            varname="default value"
        }