### Summary

This document provides the specification of Dial800’s RoundTrip v2.0 Sales Data Submission service, along with a description & examples illustrating how to properly use this service.

### RoundTrip v2.0 (RT2) Sales Data Submission Service

RT2 will provide the ability for authorized external parties (most commonly, call centers) to submit Call Sales & Disposition data to Dial800, to be associated with a specific telephone call.

This data will be made available to Dial800 clients & authorized third parties (i.e. Media Agencies) via nightly file transmissions for use in decision making, media purchase analysis, etc.

### No Encryption

All web services defined herein will communicate without SSL/TLS protocol via a regular “HTTP://” URL

### Authentication

All web services defined herein will support basic HTTP authentication, with a base-64 encoded UserName & Password. Custom headers are not to be used for authentication purposes.

For existing CallView users, their existing ID & Password may be used to gain full, unrestricted access to call data via these services.

For Call Centers servicing CallView clients, they will be issued a “restricted” set of credentials that allows them to post call sales data for calls which they have received & processed.

### Endpoints

The test/staging endpoint where these services will be available at is:

http://stage-api.roundtrip.dial800.com/roundtrip

The production endpoint where these services will be available at is:

http://roundtrip.dial800.com/roundtrip


### Timing

RT2 data submissions are performed on a synchronous basis, and will attempt to match the data submitted  to a call in the CallView databases, and will return an HTTP status code as part of the response indicating success or failure of the matching & data posting process.

Consequently, call sales data submitted via the RT2 API must be submitted AFTER the call is completed and the basic call data has been inserted into the CallView databases. In many cases, the call data will be available within CallView 5-10 seconds after the call is terminated. In some cases, there may be a delay as long as 10-15 minutes before the call data becomes available.

Any party attempting to submit RT2 data via this API must implement their submission code to accommodate the possibility that the target call data is not yet available within the system, detecting potential “Call Not Found” return codes, and waiting/delaying/queuing the RT2 API request for retry at a later time.

PLEASE NOTE: Dial800 begins nightly batch processing of data exports at 2am PT, which means all data submissions intended for extraction & reporting to a media agency or other external party must be completed prior to that time.


### Telephone Number Formatting

All Telephone numbers within this API will be formatted according to http://tools.ietf.org/html/rfc3966. However, at this time, Dial800 will only be supporting us-based (NANPA) telephone number formats; International number formats will be supported at a future time.


### RoundTrip v2: Submit Call Sales Data

Resource URI

```html
/roundtrip
```

HTTPS POST

This request is initiated by an authorized API user, and may be used to submit call sales data for calls matching the criteria provided in the request.

The RT2 data payload will be submitted in the POSTDATA of the HTTPS call. (See the Appendices of this document for valid payload XML structures representing valid RoundTrip data formats that are available for use.)

Data for a single call should be submitted with a single web request. The response to each request will provide a standard HTTP response code (i.e. “200” indicating “Success”), with the response body including the Call ID of the call which was matched.

The Call ID of the matched call ought to be saved, if possible, by the call center submitting the data, as it MUST be subsequently provided for making corrections to the RoundTrip data if the original submission is discovered to be in error.

Call Sales Data may be associated with a call in one of two ways:

1. By Search: The Caller’s Telephone Number (“ANI”), Target Number (“RingTo”), and Call Date & Time (“StartDateTime”) must be provided to identify the call to be matched. If available, the original Toll-Free Number (“TFN”) optionally may also be provided to improve the accuracy of the matching process; however, it is not required. In the unusual event that the provided criteria match more than one call within the CallView databases, the first (earliest) call representing a valid match will be the call that the Sales data is associated with.
  * a. Because the matching algorithm matches the first (earliest) call found to be a valid match, Dial800 recommends that call sales data is always submitted to the RT2 API in chronological order based upon the Call Date & Time of the call. This provides a very high degree of accuracy in matching calls this way.
  * b. Dial800 also recommends submitting an RT2 API post for each & every telephone call handled by the call center, *NOT* only sending data for calls that resulted in a sale. In addition to providing a complete & accurate set of data, this also helps the RT2 call matching algorithm behave as accurately as possible.
2. By CallID: If the Dial800 CallID is known, RT2 data may be associated with a specific call within the CallView databases by providing this key. When this key is provided, the data attributes listed in #1 (above) are not required, and identifying a specific call record can be guaranteed. This method would most commonly be used to replace previously-submitted call sales data that is found to be in error, and requires correction.


#### Example 1 - Request

Associate call sales data to a specific call by CallID.

```xml
/roundtrip

POSTDATA payload…

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

Associate call sales data to a call matching the search criteria. (Please Note: Optional “<DNIS>” element omitted!)

```xml
/roundtrip

POSTDATA payload…

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
