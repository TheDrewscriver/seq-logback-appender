# seq-logback-appender

Send logback log events to a seq server

## NOTE!
This was literally copy pasted out of other code and placed into this repo.
It still has dependencies from original project and the namespaces haven't been changed.
I'd like to make it a solid reusable library.
I know there are more generic libraries we could use as well. 

If you want to help me, that would be cool, andrew...  :)

## Usage

Add the required properties to your application.yaml file. Note, batchCount defaults to 1.
```yaml
seq:
  batchCount: 5
  apiKey: YOURAPIKEY
  server: http://URLTOYOURSEQSERVER
  port: 80
  fileName: YOURPROJECTNAME.seq.log
```


For each project, include the seq logback settings file in your main logback settings file
```xml
    <include resource="seq-logback-settings.xml" />
```

Then, you can add the appenders to your profiles. "SEQ" sends logs to server and "SEQ-FILE" logs to a file.
```xml
    <springProfile name="dev">
        <root level="INFO">
            <appender-ref ref="SEQ" />
            <appender-ref ref="SEQ-FILE" />
        </root>
    </springProfile>
```
Done...
 
 But wait, there's MORE!

## Adding more data to log
The base Seq Log Event Layout SeqLogEventLayout will map the basic logging data to the base SeqLogEntry
object and send it to the specified seq server. To add more, extend SeqLogEventLayout and SeqLogEntry to add more data.
Then set the seq layout class to use in your logback settings file.

#### Example:

1: Extend SeqLogEntry to add your fields
```java
@Getter
@Setter
public class CpeSeqLogEntry extends SeqLogEntry {

    @JsonProperty("User")
    private String user;

    @JsonProperty("Method")
    private String method;

    @JsonProperty("Pid")
    private String pid;

    @JsonProperty("Query")
    private String query;

    @JsonProperty("Referer")
    private String referer;

    @JsonProperty("IP")
    private String ip;

    @JsonProperty("Url")
    private String url;

    @JsonProperty("Token")
    private String token;

    /**
     * Constructor
     * @param eventObject The logback event
     * @param dateFormat Date format for timestamp
     */
    public CpeSeqLogEntry(ILoggingEvent eventObject, Format dateFormat) {
        super(eventObject, dateFormat);
        Map<String, String> mdc = eventObject.getMDCPropertyMap();
        this.setUser(mdc.get(Constants.LOGGED_ON_USER_NAME));
        this.setMethod(mdc.get(Constants.REQUEST_METHOD));
        this.setPid(mdc.get(Constants.ADMIN_PID));
        this.setQuery(mdc.get(Constants.REQUEST_QUERY));
        this.setReferer(mdc.get(Constants.REQUEST_REFERER));
        this.setIp(mdc.get(Constants.REQUEST_IP_ADDRESS));
        this.setUrl(mdc.get(Constants.REQUEST_URL));
        this.setToken(mdc.get(Constants.AUTH_TOKEN));
    }
}

```

2: Extend SeqLogEventLayout to use your new CpeSeqLogEntry class
```java
public class CpeSeqLogEventLayout extends SeqLogEventLayout {

    @Override
    public String doLayout(ILoggingEvent event) {
        return this.getLogEntryJsonString(new CpeSeqLogEntry(event, this.dateFormat));
    }
}
```

3: Add SEQ_LAYOUT property to your base logback settings xml file to use your new extended layout class.
Make sure the namespace to your layout is correct.
```xml
    <property scope="context" name="SEQ_LAYOUT" value="com.conversantmedia.cpeui.shared.util.seq.cpe.CpeSeqLogEventLayout"/>
    <include resource="seq-logback-settings.xml" />
```
Done. Now these new fields will be added to all your logging events. 

## Tips
Remember to use debug="true" in main logback configuration file when trying to set up.

