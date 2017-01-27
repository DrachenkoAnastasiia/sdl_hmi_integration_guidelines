## SetMediaClockTimer

Type
: Function

Sender
: SDL

Purpose
: Set a value and update mode for the media clock of a media application.

The UI.SetMediaClock timer request indicates either an initial value for the media clock timer for a media application or an update to the value. The request may come in for the application which is not currently active on the HMI.

_**Note:**_   
_SDL uses the information about:_   
- _mediaClock field_   
- _the format of time value confirmed to be supported by HMI within response to UI.GetCapabilities._   

### Request   

#### HMI must:   
**1)** perform the update type indicated by the `updateMode` parameter:   
- If the application is not active, the HMI must still store the values to be calculated for later display on the HMI.   
- If the application is active, the updates must begin immediately.   

**2)** exhibit the following behavior based on the `updateMode` parameter:
  * COUNTUP/COUNTDOWN modes:   
    * Start counting up or down from the requested `startTime` value with a step of 1 second;
    * Continue counting up or down until:
           - The next request of _SetMediaClockTimer_ with appropriate parameters comes;   
           - Zero is reached in the case of COUNTDOWN.   
  * PAUSE mode:   
    * Pause the timer that is counting up or down;   
    * If `startTime` or `endTime` parameters are provided, the values must be updated on the HMI.   
  * CLEAR mode:    
    * Clear `startTime` to 00:00:00 in the case that the `startTime` parameter is not provided in the request, otherwise, `startTime` must be updated with a new value. It is up to HMI to determine the way the media clock timer is cleared: either to remove it from display or to set it to zero.   
    _**Note:**_   
- SDL will not send this request if the `mediaClock` field is not indicated as supported in [UI.GetCapabilities].   
- HMI must remember the mode and the value (continuing to update it in case of COUNTUP / COUNTDOWN) of the media clock timer associated with appID and display the accurate values whenever the appID application is activated after having been deactivated.   
- Initially, the appID together with other application-related information is provided by SDL within one of _UpdateAppList_ and _OnAppRegistered_ RPCs.   
[UI.GetCapabilities]: https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/UI/GetCapabilities/index.md
**3)** Respond with the [result code] correspondingly to the results of this RPC execution.
[result code]: https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/UI/SetMediaClockTimer/index.md#response
**4)** SDL must transfer _SetMediaClockTimer_request_ (params, _enableSeek_) to HMI and respond with <received_resultCode_from_HMI> to mobile application after receiving this request from mobile application with:   
         - valid _enableSeek_ parameter;   
         - other valid parameters related to this request.

_**Note:**_   
_In case HMI sends invalid notification that SDL should transfer to mobile app, SDL must log the issue and ignore this notification._


#### Parameters

|Name|Type|Mandatory|Description|
|:---|:---|:--------|:---------|
|startTime|[Common.TimeFormat]|false|_startTime_ must be provided together with modes: "COUNTUP", "COUNTDOWN", "PAUSE" to HMI. _startTime_ will be ignored for "RESUME", and "CLEAR".|
|endTime|[Common.TimeFormat]|false|endTime can be provided together with modes: "COUNTUP", "COUNTDOWN", "PAUSE" to HMI. To be used to calculate any visual progress bar (if not provided, this feature is ignored). If endTime is greater then startTime for COUNTDOWN or less than startTime for COUNTUP, then the request will return an INVALID_DATA. _endTime_ will be ignored for "PAUSE", "RESUME", and "CLEAR".|
|updateMode|[Common.ClockUpdateMode]|true|Enumeration to control the media clock. In case of pause, resume, or clear, the start time value is ignored and shall be left out. For resume, the time continues with the same value as it was when paused.|
|appID|Integer|true|ID of application that requested this RPC.|
|enableSeek|Boolean|false|Defines if seek media clock timer functionality will be available (when _**DD**_ is off) allowing for touch input on the media clock timer from the user. If it is omitted, the value is set to false.|

[Common.TimeFormat]: https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/Common/Structs/index.md#timeformat
[Common.ClockUpdateMode]: https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/Common/Enums/index.md#clockupdatemode


### Response

_**Note:**_   
For detailed information, see Section [Sequence Diagrams](https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/UI/SetMediaClockTimer/index.md#sequence-diagrams). 

_Please see [Result Enumeration] for all SDL-supported codes._
[Result Enumeration]: https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/develop/docs/Common/Enums/index.md#result


#### Parameters

This RPC has no additional parameter requirements

### Sequence Diagrams

SetMediaClockTimer COUNTUP and COUNTDOWN for Full and Background applications
![SetMediaClockTimer](./assets/SetMediaClockTimerUpDownFullBackground.png)


SetMediaClockTimer Pause and Resume for Active Application
![SetMediaClockTimer](./assets/SetMediaClockTimerPauseResumeActive.png)


SetMediaClockTimer COUNTDOWN for a deactivated application
![SetMediaClockTimer](./assets/SetMediaClockTimerDownDeactivate.png)


SetMediaClockTimer with _enableSeek_ param
![SetMediaClockTimer](https://github.com/DrachenkoAnastasiia/sdl_hmi_integration_guidelines/blob/new_param_setmediaclocktimer/docs/BasicCommunication/OnSeekMediaClockTimer/assets/OnSeekMediaClockTimer.png)

### Example Request

```json
{
  "id" : 109,
  "jsonrpc" : "2.0",
  "method" : "UI.SetMediaClockTimer",
  "params" :
  {
    "startTime" :
    {
         "hours" : 0,
         "minutes" : 18,
         "seconds" : 17
    },
    "updateMode" : "COUNTUP",
    "appID" : 65146
  }
}
```
### Example Response

```json
{
  "id" : 109,
  "jsonrpc" : "2.0",
  "result" :
  {
    "code" : 0,
    "method" : "UI.SetMediaClockTimer"
  }
}
```

### Example Error

```json
{
  "id" : 109,
  "jsonrpc" : "2.0",
  "error" :
  {
    "code" : 11,
    "message" : "Invalid data",
    "data" :
    {
      "method" : "UI.SetMediaClockTimer"
    }
  }
}
```
