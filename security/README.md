Introduction - Provide a description of the product and what you were hired to do
Methodology - Describe your methodology pen testing the application, including the tools that you used
Abstract of Findings - Provide an overview of all the security and privacy issues you identified. This section should be written for non-technical managers who do not have technical expertise and do not have time to read the entire document. Write this section using lay language.
Issues Found - For each issue that you find, document:
	Issue (e.g., database injection, really bad programming practice)
	Location or page where issue was found
	Severity of issue (e.g., low, medium , or high). Justify your answer.
	Description of issue. How did you find it? A screenshot of problem is excellent.
	Proof of vulnerability. Show pictures or it didn't happen.
	Resolution. Make recommendation(s) on how issue can be resolved. Show code if possible.
Conclusion - You can also list future considerations and recommendations with costs.
References - A list of references and links that you used for your work.

Introduction
	The product is a backend for a google maps checkin service. The map displays historical locations within a mile of you, as well as users who have checked in on that service. Each time you load the page you are checked in at your current location. I was hired to find vulnerabilities in this backend, provide proof of them and recommend how to fix those vulnerabilities.

Methodology
	I tested the application by first reading through the source code for the backend, then running a static analysis on the platform using veracode, then finally exploiting the vulnerabilities to prove they are real and can be exploited.

Abstract of Findings
	The three vulnerabilties found in this program are very significant, but not very difficult to fix. The first one is storing the username and password of the database in the code. This is bad because if anybody has access to that file, they can access the database and do anything from deleting and stealing the data to changing the username and password of the account so that the original users no longer have access. The second and third issue fall under one roof. The input given from the user, is not checked at all, which basically gives them free range to hijack the service and do whatever they want with it. This means that if they wanted to it would redirect users to their own website, or even worse steal the users information without making anything in the service act differently at all.

Issues Found
1. The first issue that I found was that the credentials for the heroko account are in the index.js file in plaintext. They are in the uri for the mongoDB, which means they are also available to anybody sniffing the network. This issue is on line 6, where the MONGOLAB_URI is defined in the heroku index.js file. The severity of this is high because once anybody has the credentials to the account they can do anything: they can steal all the data, they can delete all the data or they could even change the credentials so the original user doesn't have access to the account anymore. I found the issue by reading through the source code in the index.js file for the heroku backend. The following line is the code that contains the issue
	MONGOLAB_URI='mongodb://heroku_gt907wrx:6u51ols9npis0uf54939a69ohn@ds259325.mlab.com:59325/heroku_gt907wrx'
This problem could be solved by storing an encrypted version of the credentials, for example
encryptedUser=ENCRYPTION_OF_USERNAME
encryptedPASS=ENCRYPTION_OF_PASSWORD
MONGOLAB_URI='mongodb://' + encryptedUser + encryptedPass + '59325/heroku_gt907wrx'

2. The second issue is that user input is not sanitized, and is thus succeptible to cross site scripting. The issue is on lines 71 and 101-103. The severity of this is high, because cross site scripting means that somebody can completely cripple your website, as the user sees it. Even worse, somebody could keep the website functional in all aspects, except for that sensitive user data is sent to them as well as to you. In the latter case, neither the user nor the owner of the website would be aware that anything was wrong. I found this issue by reading through the source code. The lines where this vulnerability is present are shown here:
	coll.find({login : request.query.login}).toArray(function(err, results)
	var login = request.body.login;
	var lat = request.body.lat;
	var lng = request.body.lng;
The way to fix this would be to remove any special characters, that allow script excecution from the query parameters and body parameters. Some of those characters could be "<>/\n;'(), to name a few.

3. Improper neutralization of response. On lines 50, 85 and 127. The severity is high for this issue, because while it could be very costly, as it is in a sense the second half of the injection that was mentioned in number 1. The outcome of successfully doing this attack would be that the user's browser would excecute whatever scripts the attacker has inserted into the response. This issue was found by using the veracode static scan. The way to fix this issue is to sanitize the responses that are sent back to the user, in the same way that user input is sanitized before it is sent to the server.
	response.send(indexPage);
	response.send(peoplejson);
	response.send(results);

Conclusion
	The good news is that these issues are not extremely hard to fix, nor are they expensive to. None of them are design flaws, rather they are implementation flaws, meaning they can be fixed by just changing a few lines of sources code, rather than redesiging and then subsequently reimplementing the whole program. I would say the changes could be made in a few of hours, including time to test that the service still works as intended and is no longer succeptible to the attacks that are outlined here.

References
My knowledge from this class and Comp 116, veracode.