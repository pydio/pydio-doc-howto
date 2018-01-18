##Security basics
### Architecture
Storing all your organization information in a SaaS-based Box service can prove to turn into a nightmare: are the provider’s servers located in a country that is compliant with your juridiction? Is the service provider trust-worthy? Can he actually read the data? And eventually let third-party read them (like NSA…)? And what if a trainee has opened a “box” last year, and no one can close it anymore? Will she still see all the documents forever??

Using Pydio is a simple solution to this: take back the control, by installing your enterprise box on your own servers. This does not necessarily means a server inside your office building, but simply leveraging your existing architecture. Maybe the enterprise already owns a piece of storage somewhere in a data-center, that has been negociated with a local dealer, and why should you migrate everything to another storage mechanism? Or you have set up a public-cloud based infrastructure, but on which you decide to encrypt everything, keeping away the keys from the provider, and that you access only through a VPN…

As such, by construction, Pydio provides a “passive” security model that will raise the standards in most cases.

### Open Source. Mature.
What open source guarantees, is a better quality and a more secure code: there can’t be any spyware or malware delivered with the solution, as a whole community is watching the code closely, and it would just be disclosed in no time. Furthermore, security experts regulary audit the code for vulnerabilities, and when they are found, they are disclosed through the public channels. This forces the contributors to be always listening and reactive, to avoid losing the trust of the community.

Pydio is a 5 years old project, with more than 500 000 downloads. Its maturity is a strong gage of security.

### Encryption
Encrypting the data during communication and at rest is a recommanded procedure. The first is easier to provide that the latter, and should always be set up.

By using SSL encryption ( = accessing Pydio through HTTPS instead of HTTP ), you will defeat the most easiest flaws that can happen. When connecting from a public Wi-Fi, it’s really easy to intercept the communication, and sending everything in clear is the best way to get information stolen.

For encryption Pydio currently provides an EncFS implementation, that is based on a encryption of the data on the file system, and the decryption is done only during the user session. Full client-side encryption is still a hard one, as it must be in that case provided by the HTML clients themselves (= the browser), requiring a lot of computer processing.

## Common security flaws
### Web-App protection
Pydio is created with security in mind: you will put your IP in there, you HAVE to be protected. Beyond the common authentication features and protections barriers that every software like this provides, the web application has been regularly tested and coded with the common security flaws in mind.

Below are some of the most current and how they are handled.

+ XSS: Sanitization of many parameters, included filenames and metadata.
+ CSRF: a secure token must be provided for all requests
+ Brute-force login: After three wrong logins, user must prove she’s a human by filling a CAPTCHA form
+ XPath Injection: heavily relying on XML, the registry is protected from injections by a sanitization mechanism.

### Mobile Clients
The mobile applications (iOS and Android) store the passwords using the standard “KeyChain” mechanism of each platform. They both support HTTPS connection (see below). The iOS application also provides a way to PIN-lock the whole application, this is in the roadmap for the Android version.

## How to report a vulnerability?
Write directly to security@pydio.com
