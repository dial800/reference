## RoundTrip v2.0 (RT2) Sales Data Submission Service

RT2 will provide the ability for authorized external parties (most commonly, call centers) to submit Call Sales & Disposition data to Dial800, to be associated with a specific telephone call.

This data will be made available to Dial800 clients & authorized third parties (i.e. Media Agencies) via nightly file transmissions for use in decision making, media purchase analysis, etc.

### No Encryption

All web services defined herein will communicate without SSL/TLS protocol via a regular “HTTP://” URL

### Authentication

All web services defined herein will support basic HTTP authentication, with a base-64 encoded UserName & Password. Custom headers are not to be used for authentication purposes.

For existing CallView users, their existing ID & Password may be used to gain full, unrestricted access to call data via these services.

For Call Centers servicing CallView clients, they will be issued a “restricted” set of credentials that allows them to post call sales data for calls which they have received & processed.

### Endpoints

The test/staging endpoint can be found at:

http://stage-api.roundtrip.dial800.com/roundtrip

The production endpoint can be found at:

http://roundtrip.dial800.com/roundtrip


### Timing

RT2 data submissions are performed on a synchronous basis, and attempt to match the data submitted  to a call in the CallView databases. HTTP status codes are returned as part of the response, indicating success or failure of the matching & data posting process.

Consequently, call sales data submitted via the RT2 API must be submitted AFTER the call is completed and the basic call data has been inserted into the CallView databases. In many cases, the call data will be available within CallView 5-10 seconds after the call is terminated. In some cases, there may be a delay as long as 10-15 minutes before the call data becomes available.

Any party attempting to submit RT2 data via this API must implement their submission code to accommodate the possibility that the target call data is not yet available within the system, detecting potential “Call Not Found” return codes, and waiting/delaying/queuing the RT2 API request for retry at a later time.

PLEASE NOTE: Dial800 begins nightly batch processing of data exports at 2am PT. All data submissions intended for extraction & reporting to media agencies or other external parties must be completed prior to that time.


### Telephone Number Formatting

All Telephone numbers within this API will be formatted according to http://tools.ietf.org/html/rfc3966. However, at this time, Dial800 will only be supporting US-based (NANPA) telephone number formats; International number formats will be supported at a future time.


### RoundTrip v2: Submitting Call Sales Data

A request is initiated by an authorized API user, and used to submit call sales data for calls matching the criteria provided in the request.

RT2 data payload is submitted in the POSTDATA of the HTTP call. (See the Appendices of this document for valid payload XML structures representing valid RoundTrip data formats that are available for use.)

Data for a single call should be submitted with a single web request. The response to each request will provide a standard HTTP response code (i.e. “200” indicating “Success”). The response body will include the Call ID of the matched call.


The Call ID of the matched call ought to be saved, if possible, by the call center submitting the data, as it MUST be subsequently provided for making corrections to the RoundTrip data if the original submission is discovered to be in error.

Call Sales Data may be associated with a call in one of two ways:

1. By Search: The Caller’s Telephone Number (“ANI”), Target Number (“RingTo”), and Call Date & Time (“StartDateTime”) must be provided to identify the call to be matched. If available, the original Toll-Free Number (“TFN”) optionally may also be provided to improve the accuracy of the matching process; however, it is not required. In the unusual event that the provided criteria match more than one call within the CallView databases, the first (earliest) call representing a valid match will be the call that the Sales data is associated with.
  * a. Because the matching algorithm matches the first (earliest) call found to be a valid match, Dial800 recommends that call sales data is always submitted to the RT2 API in chronological order based upon the Call Date & Time of the call. This provides a very high degree of accuracy in matching calls this way.
  * b. Dial800 also recommends submitting an RT2 API post for each & every telephone call handled by the call center, *NOT* only sending data for calls that resulted in a sale. In addition to providing a complete & accurate set of data, this also helps the RT2 call matching algorithm behave as accurately as possible.
2. By CallID: If the Dial800 CallID is known, RT2 data may be associated with a specific call within the CallView databases by providing this key. When this key is provided, the data attributes listed in #1 (above) are not required, and identifying a specific call record can be guaranteed. This method would most commonly be used to replace previously-submitted call sales data that is found to be in error, and requires correction.


#### Example 1 - Request

Associate call sales data to a specific call by CallID.

```xml
<Call xmlns="http://www.dial800.com/roundtrip/2011-07-15">
    <!—- Core Data -->
    <ID>X089341KJH941KJ2H4O985</ID>

    <!—- Supplemental Call Data Should Be Inserted Here -->
</Call>
```

####Example 1 – Response

```html
200 OK

<ID>XDhshURwv60Q2nRyN4cnGVCmMB1cP</ID>
```

An HTTP 200 response will indicate success, and will return the matched CallID data in the response body. HTTP 4xx responses will indicate failure; specifically, a “404 Not Found” status will indicate that a call record matching the input criteria could not be located.

####Example 2 - Request

Associate call sales data to a call matching the search criteria. (Please Note: Optional “&lt;DNIS&gt;” element omitted!)

```xml
<Call xmlns="http://www.dial800.com/roundtrip/2011-07-15">
    <!—- Core Data -->
    <ANI>tel:3101234567</ANI>
    <Target>tel:8881234567</Target>
    <CallStart>2010-09-01T09:13:31-07:00</CallStart>

    <!—- Supplemental Call Data Should Be Inserted Here -->
</Call>
```

####Example 2 – Response

```html
200 OK

<ID>XDhshURwv60Q2nRyN4cnGVCmMB1cP</ID>
````

----

## Appendix “A”: Basic Call Sales Data

### POSTDATA XML payload structure

```xml
<Call xmlns="http://www.dial800.com/roundtrip/2011-07-15">
    <!—- Core Data -->
    <ID>X089341KJH941KJ2H4O985</ID>
    <DNIS>tel:8005555555</DNIS>
    <ANI>tel:3105555555</ANI>
    <Target>tel:3109999999</Target>
    <CallStart>2011-07-15T01:02:03-08:00</CallStart>

    <!—- Supplemental Form-Based Data Should Be Inserted Here -->
</call>
```

Where the XML elements provided in the POSTDATA of the request body are as follows:

Parameter Name
 

Description

ID
	

String value representing the alphanumeric Call ID of the phone call to be matched for the associated Sales data. [OPT]

DNIS
	

10-Digit string value representing the DNIS (TFN) of the phone call to be matched for the associated Sales data. [OPT]

ANI
	

10-Digit string value representing the ANI (Caller #) of the phone call to be matched for the associated Sales data. [OPT]

Target
	

10-Digit string value representing the Target (“RingTo”) number of the phone call to be matched for the associated Sales data. [OPT]

CallStart
	

The Call Start Time representing when this call was initiated. This value must be expressed using the standard XML DateTime format which includes the timezone offset identifier(i.e. “YYYY-MM-DDThh:mm:ss±HH:MM” or “YYYY-MM-DDThh:mm:ssZ”). [OPT]

	

Supplemental Attributes
	

Refer to subsequent document appendices and/or addendum documents for partner-specific XML structures & examples of additional data which may be posted.

PLEASE NOTE:

    If the “<ID>” element is used, no other “Core Data” parameters are required, nor should they be passed, or an error will be issued. The “<ID>” element must always be passed on its own.
    If the “<ID>” element is not passed/used, then the “<ANI>”, “<Target>”, and “<CallStart>” elements will be required to be passed, and the “<DNIS>” element may optionally be passed.