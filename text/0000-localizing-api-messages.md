
* Start Date: 2022-03-29
* RFC PR: 
* FOLIO Issue: 
* Submitter : Radhakrishnan Gopalakrishnan (rgopalakrishnan@ebsco.com)
* Co-Submitter(s) : Zak Burke (zburke@ebsco.com)
* Sub Group : 
  * Marc Johnson (marc.johnson@k-int.com)
  * Julian Ladisch (julian.ladisch@gbv.de)
  * Peter Murray (peter@indexdata.com)
  * Tod Olson (tod@uchicago.edu)

# Localizing API (Backend) Messages

## Summary
  The purpose of this RFC is to define an approach that allows API developer to provide
messages in the API response depending on the language preference set in the request. 


## Motivation

- Help users understand the information coming from the application backend using a language they understand better
- To allow FOLIO platform to be used by users across the world 
- To create a tremendous opportunity for growth that we can never could have achieved within just one country.
- To Promote consistency and make it easier for tools and translators when handling translation files

### In Scope Requirements/Use cases
- Return localized messages based on the value passed in the accept-language header
- Handle static messages with placeholder(s)

### Out of Scope Requirements/Use cases
- [Plural Syntax](https://wiki.folio.org/display/I18N/How+To+translate+FOLIO#HowTotranslateFOLIO-Pluralsyntax)
- Naming convention to be used for keys
- Process for managing (decoupled from lokalise.com) translations
- Returning formatted (HTML/Markdown) messages
- Usage of [soft hyphen](https://wiki.folio.org/display/I18N/How+To+translate+FOLIO#HowTotranslateFOLIO-Softhyphentobreakwords) to break messages
- Support customization per tenant
- Process  for [back porting](https://wiki.folio.org/display/I18N/Backport) API/Backend messages similar to what we have for the front end 
- Support for [controlled vocabulary](https://issues.folio.org/browse/UXPROD-3148) (runtime data/data coming from DB tables. E.g. Patron Groups for tenants)

## Detailed Explanation/Design
* Translation file format will depend on the language that is used to develop the module
  * Java, Groovy - properties
  * JavaScript, nodejs - json
  * Any other language - Please contact Technical Council for further guidance
* Translation files MUST be placed under translations/\<Backend Module Name\>
* Translation file name should be ll_CC.ext. Here ‘ll’ is an ISO 639 two-letter language code, and ‘CC’ is an ISO 3166 two-letter country code.
* Translation message keys MUST be a string (alphanumeric characters ONLY)
* Developer should follow language/framework specific recommended best practices for loading translation at runtime
* When handling messages with placeholders, replace the placeholders with the actual value on the server side before
  sending it back to the client
* STOP using the lang query string parameter.
* accept-language header value MUST be in ll_CC format. , where ll is a two-letter language code, 
  and CC is a two-letter country code.
* When accept-language header is missing or has an incorrect value, return the message value in en_US. If message value 
  cannot be found, return the message key


## Rationale and Alternatives
<span style="color:green">**_Following 4 options were considered. All options except Option 4 involves stripes in some manner.
Coupling the API backend to a specific front end framework limits the usage of FOLIO API to
only clients that are built using Stripes. As a result, we are recommending that we go with Option 4._**
</span>.

### Option 1
Distribute them over the existing front-end modules:

ui-checkin with translation key ui-checkin.mod-circulation.itemNotFound
ui-requests with translation key ui-requests.mod-circulation.patronHasItemOnLoad

*Pros*
- Uses existing front-end modules

*Cons*
- Cannot disable a front-end module for a tenant if it hosts a mod-circulation translation key that is needed by another enabled front-end module.
- Not all originating messages can be mapped to a frontend module 

### Option 2a
For each back-end module with translation keys create a dedicated front-end module with language files:

For example create ui-mod-circulation module, and use translation keys

ui-mod-circulation.itemNotFound
ui-mod-circulation.patronHasItemOnLoad

*Pros*
- Any front-end module that uses mod-circulation can require ui-mod-circulation to load the translation files.
- All mod-circulation translation strings go into a single module.
*Cons*
- Additional overhead in managing more ui modules
- Not all originating messages can be mapped to a frontend module

### Option 2b
For each back-end interface with translation keys create a dedicated front-end module with language files:

For example create i18n-circulation module, and use translation keys

i18n-circulation.itemNotFound
i18n-circulation.patronHasItemOnLoad

*Pros*
- Any front-end module that uses the circulation API can require i18n-circulation to load the translation files.
- All circulation translation strings go into a single module.
*Cons*
- Additional overhead in managing more ui modules
- Not all originating messages can be mapped to a frontend module

### Option 3
The back-end module hosts the language files.

mod-circulation creates a translations/mod-circulation/ directory.

Stripes fetches the translation files from the back-end module and can process them the same way as translation files from frone-end modules.

*Pros*
- Software and language files are in the same repository. Options 1 and 2 require to add the message string to some other repository using a second pull request.

*Cons*
- Is this technically feasible? How can Stripes fetch languages files from a non-Stripes module?

### Option 4:
When calling the back-end pass the required locale. The back-end maintains the translations files, replaces the placeholders and puts the final string into the "message" property. A lang query parameter that many FOLIO API interfaces already have or an "accept-language" header line might be used.

*Pros*
- Java code and translation files live in the same repository. No work for the front-end.
- This is a well known pattern in the Java community

*Cons*
- None

## Risks and Drawbacks

None that we can think of. The pattern we are adopting is a standard pattern in the java community


## Unresolved Questions
- Use of ICU for standardizing messages value format in translation files
- Tradeoffs 
- Can the localization project support translating files that are in different formats. 
  JSON, properties file etc..
