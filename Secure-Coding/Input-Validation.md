
Input Validation
Hello everyone, let's discuss on Input Validation


In this section, let's understand what input validation is from a security engineer perspective and its types.

Numerous applications require users to provide input, such as email addresses and passwords for authentication or personal details and credit card numbers for purchasing products or services. However, users may unintentionally or intentionally provide incorrect inputs, leading to potential usability or security issues within the application. Therefore, it is crucial for applications not to rely solely on user inputs.

Input validation is a security measure that can help protect applications from invalid or malicious inputs. It involves testing the input to ensure it aligns with the application's standards before trusting and using it. Applications can reject invalid inputs to minimize their vulnerability to injection attacks, including SQL injection and command injection. These attacks deliberately use malformed inputs to trick the application into interpreting user-provided data as commands, causing the application to malfunction.

Before diving into the different types of input validation, lets discuss the best practices of input validation.

●	Resolve/Decode before validation: Inputs like URLs and file paths may need to be resolved or decoded before use to avoid potential issues.
●	Perform Server-Side Validation: Input validation should be performed on the server-side as client-side validation can be bypassed with web proxies.
●	Validation is the first line of defense: While input validation can help filter malicious inputs, it should not be the only defense against attacks like cross-site scripting (XSS).
●	Reject, don't scrub: Invalid inputs should be rejected rather than trying to scrub them as malicious inputs can take advantage of scrubbers, leading to negative consequences.

Let's discuss the different ways we can perform input validation.

●	Blocklists
●	Allowlists
●	Syntactic Validation
●	Semantic Validation

In this context, we are discussing only Blocklists and Allowlists.


●	Blocklists
	
○	Introduction to Blocklists.

Blocklists are a type of input validation mechanism that is widely used to prevent malicious inputs in applications. They work by defining a set of values that are considered harmful or unwanted, such as specific words, phrases, or characters. When a user tries to submit an input that matches any of the values on the blocklist, the application will reject it.

In most cases, blocklists are created based on the reserved characters or commands of a particular programming language or application. For example, single or double quotation marks are commonly used to denote strings in programming languages, but they can also be used to inject malicious code into an application. Therefore, a blocklist may include these characters to prevent users from using them in their inputs.

Let’s consider  the following SQL Query as an example:


SELECT * FROM users WHERE username='<user>'.

Suppose we are given the ability to enter any username we desire as a user. If we decide to enter a malicious username such as "a' OR '1'='1", the query generated from it would be:


SELECT * FROM users WHERE username='a' OR '1'='1'

The query produced by the malicious input has a considerably altered meaning compared to the original query. It will return a record if it satisfies one of two conditions.

username = a

Or


1=1

It is crucial to note that the second condition in the modified query is always true, which implies that all the records in the database will be returned. This could potentially expose sensitive information to unauthorized users.

In this scenario, we exploited the fact that the query uses a single quote character to differentiate between data and commands. By incorporating a single quote character in our input, the OR was interpreted as a command.

To prevent such attacks, an application can use a blocklist that includes a single quote character. This would cause inputs such as a' OR '1'='1 to be rejected before they are added to the SQL query.

However, this approach is excessively restrictive and may cause valid inputs to be rejected. A more effective solution is to use parameterized queries, also known as prepared statements. This technique involves separating the data from the commands in the SQL query and using placeholders for the user input. This way, the user input is treated as data and not as commands, which eliminates the risk of SQL injection attacks.

●	Building a Blocklist

When designing a blocklist, it is essential to strike a balance between being overly restrictive and protective. One common approach is to start with the list of reserved characters for a particular language or data type. However, it's important to recognize that some parts of the data may be misinterpreted by the application, and some reserved characters may not be a threat.

For example, in SQL queries, a blocklist that contains all reserved characters will be overly restrictive, as some characters may be part of legitimate user input. Instead, the focus should be on blocking characters that could prematurely terminate a quoted section.

Moreover, a blocklist doesn't necessarily have to be limited to single characters. In HTML, tags like <script> and </script> mark different types of content embedded within the HTML. While < and > define HTML tags, they can also be a part of legitimate user input. Therefore, including entire tags like <script> and </script> in a blocklist can effectively prevent malicious inputs with embedded executable code.

Let’s review a sample code

blocklist = ["'","\"","--","select","union"]
def validate(input):
    for token in blocklist:
        if token in input.lower():
            return False
    return True

data = input("Enter input: ")
res = validate(data)
if res:
    print("Input accepted.")
else:
    print("Invalid input.")

The code snippet in Python presented above showcases the usage of the input() function to receive user input, which is then passed to the validate() function implementing the blocklist. This validate() function checks each token in the blocklist (comprising of single and double quote characters) and verifies if they exist in the user input by utilizing the in operator of Python. If there is a match, the input gets rejected. However, if the loop concludes without any matches, the input is accepted.

●	Limitations of Blocklists

While blocklists can be a helpful tool in preventing certain types of malicious input, they are not foolproof and can have unintended consequences. For example, commonly used characters such as single and double quotes may be necessary for legitimate user input. Additionally, blocklists may not cover all possible ways that malicious code could be injected.

In some cases, it may be more effective to use a whitelist approach, where only certain types of input are allowed. This can be more restrictive, but it provides greater control over what input is accepted.

Another alternative is to use a combination of both approaches, such as a hybrid approach. This could involve using a blocklist for known malicious inputs and a whitelist for known legitimate inputs, while also allowing for some flexibility for unknown inputs.

Ultimately, it is important to carefully consider the specific use case and potential attack vectors when implementing input validation. A thorough understanding of the language and potential vulnerabilities can help ensure that the validation approach is effective and does not inadvertently block legitimate input.


Let's consider the following command where an attacker uses image tags instead of script tags:


<IMG SRC="javascript:alert('1');">

The input provided in the example would evade the blocklist as it does not include the <script> or </script> tag, allowing it to be embedded in the HTML code and create a pop-up window with the text "1". Although it could be remedied by modifying the blocklist, it is challenging to anticipate every possible way an attacker could circumvent the blocklist. Thus, blocklists are not a reliable approach for input validation.


●	Allowlists

○	Introduction to Allowlists.
In the previous section, we discussed blocklists that specify which characters are not allowed in user input. In contrast, an allowlist determines the characters that are permitted in user input. For instance, when a user is prompted to enter a phone number in an input field, the corresponding allowlist might allow for numeric characters (0-9) and certain special characters that are frequently used in phone numbers, like +()-. If a user tries to enter any other characters, their input will be rejected.


●	Building an Allowlist
When creating an allowlist, it's essential to ensure that all valid characters that a user might input are included. The type of data that users may input should be taken into consideration when designing an allowlist.

Examples of well-defined and suitable data types for allowlists are phone numbers, Social Security Numbers, payment card numbers, and driver's license numbers. A phone number, for instance, can include numeric characters (0-9) and certain special characters (such as hyphens and parentheses). Payment card numbers are typically numeric, potentially with spaces, while driver's license numbers are usually alphanumeric (A-Z, 0-9).


def validatePaymentCardData(input):
    input = input.replace(" ","")
    if input.isnumeric():
        return True
    else:
        return False

In the validatePaymentCardData() function, two helper functions are used to simplify the creation of an allowlist. The replace() function is used to remove optional spaces from a payment card number, while isnumeric() is used to validate that the remaining characters are all numbers. Although this code ensures that only the right character set is used, it does not validate whether the provided payment card number is correct. For example, an input of 12345 would return "True". To perform this kind of validation, Syntactic Validation is required, which will be discussed in the next Learning Unit.

Designing an allowlist for less-structured data can be challenging. Consider the example of creating an allowlist for street addresses. At a minimum, alphanumeric characters (A-Z, a-z, and 0-9), and spaces should be allowed. However, a street name could also contain certain special characters, such as a single quote (') or a hyphen (-). Such an allowlist could likely manage most English addresses assuming that the address is not abbreviated (which would make a period a valid character). Nonetheless, if the address contains unusual characters (e.g., characters from a different language), the application would reject it.

When creating an allowlist, it's essential to consider all possible valid inputs. For this reason, it's better to use allowlists for well-defined data types.



●	Limitations of Allowlists

An allowlist is a mechanism that restricts the user input to a specific set of characters or tokens. It is easy to define an allowlist for structured data since the format and contents of the data are well-defined. However, creating an allowlist for unstructured data, such as passwords, is difficult since it is challenging to predict what characters the user might include. Although a minimal restriction on the password's content is possible by disallowing dangerous characters, this may prevent legitimate user passwords.

A similar issue arises while creating an allowlist for the user's name field. Including an apostrophe or hyphen in the allowlist to account for names like O'Connor or Smith-Staton weakens its effectiveness. Additionally, including characters from other languages may not be sufficient as some names may contain unusual characters like the dollar sign ($). Ultimately, the allowlist for a name field is virtually ineffective as it includes almost every printable character.



●	Syntactic Validation

○	Intro to Syntactic Validation

Structured data, such as credit card numbers, typically follows a well-defined format. For example, a credit card number may consist of 16 digits separated into groups of four (e.g., 1234 5678 9012 3456). Syntactic validation is a technique used to verify that the data is structured correctly and of the correct type. For example, if an input field is expecting a credit card number, any input that does not consist of exactly 16 digits (or 15 digits for American Express) should be rejected as invalid input.


●	Data Type Check

To begin syntactic validation, it is important to confirm that user input matches the expected data type. For instance, when collecting a credit card number, inputs containing non-numeric characters, except for spaces, should be considered invalid.

Most programming languages provide built-in functions for data type checks, such as ctype_alpha(), ctype_digit(), and ctype_alnum() in PHP. If these functions are not available, we can attempt to convert the user input string into the correct data type. For example, if a call to Python's float() function returns a value, we know the input is a float, and any non-numeric characters will cause the function to throw an exception, indicating the input should be rejected.

●	Character Check

When it comes to verifying whether a value is a number, the process is straightforward. However, when dealing with string-type data, additional syntactic validation may be required. Take, for example, a user entering data encoded with Base64, like a blockchain address. The output of the Base64 encoding algorithm is typically alphanumeric with a few special characters (/+=).
Syntactic validation for this type of data should ensure that the input is restricted to this character set. If any other characters are present, they should be rejected.

To perform a character check, we can use either allowlists or blocklists, as discussed in previous Learning Units. In most cases, an allowlist is the better option since the data undergoing a character check is likely well-structured, making it easier to define the acceptable character set.

●	Format Check

Ensuring the correct formatting of user input is important for syntactic validation, even if the input appears correct. Regular expressions (regexes) can be used to define the structure and content of acceptable data.
Regexes can be simple, using Boolean or combining multiple acceptable values. They also have special characters with specific meanings, such as "." to match any character except line feed, "^" to indicate the start of the string, and "$" to indicate the end of the string.
Square brackets [] provide a choice between options, and hyphens can define a range of alphanumeric values. The carat (^) within square brackets means we want to match anything within the square brackets, excluding the characters.
We can also define a regex to match a certain count of a specific character, such as "?" for 0 or 1 instances, "*" for 0 or more instances, "+" for 1 or more instances, "{a}" for exactly a instance, and "{a,b}" for between a and b instances.

Here are some examples of regexes for validating specific types of data:

●	Phone Number: [0-9]{3}-?[0-9]{3}-?[0-9]{4} matches phone numbers in the form of 123-45-6789 with or without dashes
●	Social Security Number: [0-9]{3}-?[0-9]{2}-?[0-9]{4} matches an SSN with or without dashes
●	US Postal Code: [0-9]{5}(-[0-9]{4})? matches a 5 or 9 digit ZIP code, with or without a hyphen and additional 4-digit extension.
JavaScript provides the RegExp object, which enables us to define and test regular expressions. Consider the following code snippet that creates a RegExp object and uses it to check if a string contains a valid phone number:


input = "asdfasdf"
var phone = new RegExp("^[0-9]{3}-?[0-9]{3}-?[0-9]{4}$")
if (phone.test(input)){
    alert("Phone is valid")
}else{
    alert("Phone is Invalid")
}

The code shown defines a regular expression using the RegExp object in JavaScript and then uses it to validate whether a string contains a phone number.

The expression is stored in the variable 'phone', and the test() method is used to check if a given input string matches the expression. If the input matches, the method will return 'true', and an alert will display a message saying the input is valid. Otherwise, it will return 'false', and an alert will show a message saying the input is not valid.

●	File Check
File uploads to a server can be a security risk if untrusted files are allowed. Malicious code or intentionally malformed files can compromise the application. To mitigate these risks, uploaded files must be validated. Simply checking the file extension is not enough as an attacker can change the extension of a malicious file to an accepted format. Validation checks should verify that the uploaded file is a properly formatted example of an acceptable file format.

File validation difficulty varies based on the type of file being uploaded. Text files like HTML should be properly tagged and have the appropriate content within each tag. Plain text standards like JSON and XML are easier to interpret and validate than encoded formats such as Windows Executables and Zip Archives.

Manual validation is possible but using a syntax validator, if available for the format, is preferred. For example, to check if an uploaded file is in JSON format, the file can be parsed. Valid files will parse successfully, while invalid ones will cause an error or exception.

The Python code below provides an example of how to verify that a file is in JSON format:

import json
def is_json(file):
    try:
        json_object = json.load(file)
    except ValueError as e:
        return False
    return True


The Python code uses the json.loads() function to attempt to parse the provided string as a JSON object. If the parsing is successful, the function returns the parsed object indicating that the data is in valid JSON format. However, if the parsing fails, an exception is thrown which indicates that the data is not valid JSON and is rejected.


●	Semantic Validation

●	Intro to Semantic Validation

Semantic validation checks if the data entered into a field is logical, whereas syntactic validation only ensures that the data is of the right data type. For example, in an event scheduling application that requires a start and end time, syntactic validation only checks if both times are represented as numbers. On the other hand, semantic validation checks if the start time is before the end time and if the hour is less than or equal to 12/24 and the minute is less than 60.
Semantic validation is more complex because it validates whether a value "makes sense," rather than just checking if it "seems right" like in syntactic validation.

●	Range Checking
Semantic validation can include range checking, which involves verifying that a value entered by a user falls within an acceptable range of values. For instance, the check for valid times mentioned earlier involves range checking. Depending on the clock format used, a valid value for the hour falls within the range of 1-12 or 0-23, while valid minute values fall within the range of 0-59.
Range checking can be implemented using conditional statements like if/else. Here's an example code snippet that checks if a given number falls within a certain range:

def validateTime(hour,minute):
    if hour < 1 or hour > 12:
        return False
    if minute < 0 or minute > 59:
        return False
    return True

The function validateTime() demonstrated above performs range checking on the values provided for hours and minutes, ensuring that they are within acceptable ranges for a 12-hour clock. If the input is valid, the function returns True. Otherwise, it returns False to indicate that the input is invalid. While the implementation above uses an if/else statement, there are more concise and efficient ways to perform range checking in some programming languages.

For an example, let's review an equivalent Python command which checks if an hour value is valid for a 12-hour clock:

if hour in range(1,13)


The range() function in Python returns a sequence of numbers between the specified start and stop values. In the code block above, we're using this function along with the in operator to check if the provided hour value is valid for a 12-hour clock. The hour value is checked to see if it exists within the sequence returned by the range() function with start value 1 and stop value 13. Since the stop value is not included in the sequence, the range() function returns the numbers 1 through 12. The code that checks the minute value could also be rewritten to use the range() function.



●	Date Checking
Sometimes, additional checks beyond range checking are necessary to ensure that a numeric value is valid. Let's consider the following code sample, which checks if a user-provided date is correct.


def validateDate(month,day):
    if month < 0 or month > 12:
        return False
    if day < 0 or day > 31:
        return False
    return True



The previous code sample for date validation has a flaw, assuming that every month in the calendar is exactly 31 days long, which is incorrect. Hence, it could result in accepting some invalid dates. In order to validate the start and end dates, we need to check that the dates are correct relative to one another. Here is an alternative code sample that checks if a start date is earlier than the end date:


def validateRelativeDates(startMonth,endMonth,startDay,endDay):
    if startMonth == endMonth:
        if startDay <= endDay:
            return True
        elif startMonth > endMonth:
            return True
    return False

To determine if a range of dates is valid, the start date must be earlier than the end date, either in an earlier month or in the same month. The provided code sample checks both of these conditions to validate the range of dates.




●	Email Checking
Semantic validation can be challenging for some types of data, such as email addresses. While email addresses often follow a simple pattern of user@domain.com, validating them using regular expressions can be tricky. For instance, a regex like ^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+.[a-zA-Z0-9-.]+$ may work for most emails, but it misses many email addresses that are considered valid according to RFCs 822 and 5322. Building a regex to validate against these RFCs is extremely difficult, and the best examples available are complex and make simplifying assumptions.
	Fortunately, RFC 822 was superseded by RFC 5322, which provides a more comprehensive standard for email addresses. While validating against RFC 822 is nearly impossible, the following regular expression can be used to validate against RFC 5322:


(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9]))\.){3}(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9])|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])


A more effective and simpler way to validate an email address that meets the RFC's requirements is to use a built-in function such as filter_var() in PHP. This function verifies whether a user-provided input is a valid email address without requiring a complex and error-prone regex. However, it is essential to note that such methods only ensure that the email address is valid and not necessarily owned by the user. Therefore, the best approach for email address validation is to send an email to the provided address with a link for the user to "click to validate" their email address.
