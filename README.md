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

#### OBSERVATIONS ON MATCHING:
You may match a SEARCH submission ONLY ONCE. After a match is established and recorded, any futher attempts to match the same call will fail: the call has already been recorded as matched. However, you may send an UPDATE (based on Call ID value) to confirm your submissions.

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

<table cellpadding="0" cellspacing="0" class="c27">
<tbody>
    <tr>
        <th>Parameter&nbsp;Name</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>ID</td>
        <td>String value representing the alphanumeric Call ID of the phone call to be matched for the associated Sales data. [OPT]</td>
    </tr>
    <tr>
        <td>DNIS</td>
        <td>10-Digit string value representing the DNIS (TFN) of the phone call to be matched for the associated Sales data. [OPT]</td>
    </tr>
    <tr>
        <td>ANI</td>
        <td>10-Digit string value representing the ANI (Caller #) of the phone call to be matched for the associated Sales data. [OPT]</td>
    </tr>
    <tr>
        <td>Target</td>
        <td>10-Digit string value representing the Target (“RingTo”) number of the phone call to be matched for the associated Sales data. [OPT]</td>
    </tr>
    <tr>
        <td>CallStart</td>
        <td>The Call Start Time representing when this call was initiated. This value must be expressed using the standard XML DateTime format which includes the timezone offset identifier (i.e. "YYYY-MM-DDThh:mm:ss±HH:MM” or “YYYY-MM-DDThh:mm:ssZ”). [OPT]</td>
    </tr>
    <tr>
        <td>Supplemental Attributes</td>
        <td>Refer to subsequent document appendices and/or addendum documents for partner-specific XML structures &amp; examples of additional data which may be posted.</td>
    </tr>
</tbody>
</table>
	

PLEASE NOTE:

* If the “&lt;ID&gt;” element is used, no other “Core Data” parameters are required. An error will be issued is they are passed. The “&lt;ID&gt;” element must always be passed on its own.
* If the “&lt;ID&gt;” element is not passed/used, the “&lt;ANI&gt;”, “&lt;Target&gt;”, and “&lt;CallStart&gt;” elements are required. The “&lt;DNIS&gt;” element may optionally be passed.

##### &lt;DNIS&gt;

This element represents the original toll-free number dialed by the caller, and should *not* be confused with a toll-free transfer number that would be used to provide routing/access to a call center – generally speaking, the number which provides access to the call center would be considered the <Target> element.

As noted above, the <DNIS> element is optional; in the case where this number is not known to a call center or other API user at the time sales data is posted, it may be omitted (the element may be omitted entirely, or included with a blank value). If this data value *is* known at the time of data submission, Dial800 strongly recommends including it in the web request, as the presence of this data element will serve as an additional matching criteria to ensure the best, most accurate call matching possible.

##### &lt;CallStart&gt;

Please note that the “<CallStart >” value is crucial to correctly identifying and matching sales data to calls, as RoundTrip enforces a matching tolerance of ±900 seconds (15 minutes) to allow for minor variations in clock time synchronization between Dial800 and consumers of this service. If the consumer-supplied “<CallStart>” value varies from the “CallStartTime” value within CallView by more than this 15 minute tolerance, the sales data will not be matched to the call.

Also, this value must be supplied in an XML time format that specifies the timezone offset (either “Z” or “±HH:MM”). It is the responsibility of the consumer of this API to make sure that the timezone offset is expressed correctly, otherwise the start time provided via the web service request is almost certain to vary from the call data by more than 15 minutes, resulting in a “No Match” condition.

NOTE: Pay special attention to differences due to “Standard Time” versus “Daylight Savings Time” – this is a common mistake that results in the intended time being off by one hour, resulting in a “No Match” condition. 

-----

## Appendix “B” – “Classic” Dial800 RoundTrip XML Sample Data Structure

__PLEASE NOTE: Use “Content-Type: application/roundtrip.sales” in HTTP POST Request Headers!__

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Call xmlns="http://www.dial800.com/roundtrip/2011-07-15"
      xmlns:rs="http://www.dial800.com/roundtrip-sales/2011-08-04">
     <!-- Core Data Per Appendix “A”, above ... -->        
     <ANI>tel:3105555555</ANI>
     <Target>tel:3109999999</Target>
     <CallStart>2011-07-15T01:02:03-08:00</CallStart>

     <!—- Supplemental Form-Based Data Should Be Inserted Here -->
     <rs:Order payment="amex">
		<rs:Item price="100.00">OVEN</rs:Item>
		<rs:Item price="100.00">SPK</rs:Item>
		<rs:Item price="59.72">ERK 3 PAY</rs:Item>
      </rs:Order>
</Call>
```

Please Note: the supplemental form data is based on Dial800 Native API, which you can find [here](http://docs.dial800.com).

-----

## Appendix “C” – CallView360 view of RoundTrip Sales Data

Sales data provided through the RoundTrip web service interface and matched to CallView360 calls will render in the CallView360 UI.

<img src="http://c14986898.r98.cf2.rackcdn.com/Reference_RoundTripSample.png">

The “IsSale”, “Amount”, and “Item” attributes from the RT2 data submission will be derived differently depending upon the specific form data format submitted, and will render directly on the CallView UI as shown above. All other data elements will be contained in the CallView databases & provided via data transmission to the client and/or Media Agency, but will NOT render on the CallView UI.

Please Note: Sales data for these calls will be matched in a sequential (chronological) order, with the first call on line #11 matched first, and the second call on line #12 matched second. The 15 minute time tolerance still applies to these calls.