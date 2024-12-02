# Zoho-CRM
Various functions used within Zoho CRM projects

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

