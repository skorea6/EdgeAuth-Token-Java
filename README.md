<div style="text-align:center"><img src=https://github.com/AstinCHOI/akamai-asset/blob/master/edgeauth/edgeauth.png?raw=true /></div>


## Example
```java
import com.akamai.edgeauth.EdgeAuth;
import com.akamai.edgeauth.EdgeAuthBuilder;
import com.akamai.edgeauth.EdgeAuthException;


public class EdgeAuthExample {
  public static void main(String[] args) {
    String hostname = "YourAkamaizedHostname";
    String ET_ENCRYPTION_KEY = "YourEncryptionKey";
    long duration = 500L; // 500 seconds

    // Below examples here ..
  }
}
```

* ET_ENCRYPTION_KEY must be hexadecimal digit string with even-length.
* Don't expose ET_ENCRYPTION_KEY on the public repository.


#### URL parameter option
```java
try {
  EdgeAuth ea = new EdgeAuthBuilder()
      .key(ET_ENCRYPTION_KEY)
      .windowSeconds(duration)
      .escapeEarly(true)
      .build();

  /******** 
  1) Cookie 
  *********/
  String path = "/akamai/edgeauth";
  String token = ea.generateURLToken(path);
  String url = String.format("http(s)://%s%s", hostname, path);
  String cookie = String.format("%s=%s", ea.getTokenName(), token);
  // => Link or Request "url" /w "cookie"

  /************** 
  2) Query String 
  ***************/
  String path = "/akamai/edgeauth";
  String token = ea.generateURLToken(path);
  String url = String.format("http(s)://%s%s?%s=%s", hostname, path,
    ea.getTokenName(), token);
  // If url has a query string which isn't for the token, be aware of the string formatter and symbol(? and &).
  // => Link or Request "url" /w Query string
} catch (EdgeAuthException e) {
  e.printStackTrace();
}
```

* 'Escape token input' option in the Property Manager corresponds to 'escapeEarly' in the code.  
    Escape token input (on) == escapeEarly (true)  
    Escape token input (off) == escapeEarly (false)
* In [Example 2] for Query String, it's only okay for 'Ignore query string' option (on).
* If you want to 'Ignore query string' option (off) using query string as your token, Please contact your Akamai representative.


#### ACL(Access Control List) parameter option
```java
try {
  EdgeAuth ea = new EdgeAuthBuilder()
      .key(encrpytionKey)
      .windowSeconds(duration)
      .build();

  /******************
  3) Header using '*' 
  *******************/
  String acl = "/akamai/edgeauth/list/*"; //*/
  String token = ea.generateACLToken(acl);
  String url = String.format("http(s)://%s%s", hostname, "/akamai/edgeauth/list/something");
  String header = String.format("%s: %s", ea.getTokenName(), token);
  // => Link or Request "url" /w "header"

  /************************* 
  4) Cookie Delimited by '!'
  **************************/
  String acl2[] = { "/akamai/edgeauth", "/akamai/edgeauth/list/*" };
  String token = ea.generateACLToken(acl2);
  String url = String.format("http(s)://%s%s", hostname, "/akamai/edgeauth/list/something2");
  String cookie = String.format("%s=%s", ea.getTokenName(), token);
  // => Link or Request "url" /w "cookie"
} catch (EdgeAuthException e) {
  e.printStackTrace();
}
```

* ACL can use the wildcard(\*, ?) in the path.
* Don't use '!' in your path because it's ACL Delimiter.
* Use 'escapeEarly=false' as default setting but it doesn't matter turning on/off 'Escape token input' option in the Property Manager


## Usage

#### EdgeAuth, EdgeAuthBuilder Class

| Parameter | Description |
|-----------|-------------|
| tokenType | Select a preset. (Not Supported Yet) |
| tokenName | Parameter name for the new token. [ Default: \_\_token\_\_ ] |
| key | Secret required to generate the token. It must be hexadecimal digit string with even-length. |
| algorithm  | Algorithm to use to generate the token. ("sha1", "sha256", or "md5") [ Default: "sha256" ] |
| salt | Additional data validated by the token but NOT included in the token body. (It will be deprecated) |
| ip | IP Address to restrict this token to. (Troublesome in many cases (roaming, NAT, etc) so not often used) |
| payload | Additional text added to the calculated digest. |
| sessionId | The session identifier for single use tokens or other advanced cases. |
| startTime | What is the start time? (Use EdgeAuth.NOW for the current time) |
| endTime | When does this token expire? endTime overrides windowSeconds |
| windowSeconds | How long is this token valid for? |
| fieldDelimiter | Character used to delimit token body fields. [ Default: ~ ] |
| aclDelimiter | Character used to delimit acl. [ Default: ! ] |
| escapeEarly | Causes strings to be url encoded before being used. |
| verbose | Print all parameters. |

#### EdgeAuth Static Variable
```java
public static final Long NOW = 0L; // When using startTime, 0L means "from NOW".
```

#### EdgeAuth's Method
| Method | Description |
|--------|-------------|
| generateURLToken(String url) | Single URL path. |
| generateACLToken(String acl) | Single URL path - can use the wildcard (*, ?) |
| generateACLToken(String[] acl) | Multi URL paths - can use the wildcard |

Returns the authorization token string.

