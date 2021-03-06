## WOSE Glasses
WOSE glasses is a 3-device set of elements that allows you to monitor sleep phases.

When in a middle of a REM phase WOSE induces a disturbance on user sleep, if user awakes the possibilities of entering a Lucid Dream phase are likely high.

Wearable devices are controlled by a native APP in bluetooth range, which sends collected data to an external API.

Later APP lets user interact with his sleeping experience, dream memories and check previous experiences on a private dashboard.

## Key Aspects
1. Functional specs
1. System architecture
1. Technological stack
1. Testing
1. Deploy
1. Development plan
1. Profit


## Functional specifications
Product has 3 main components 
#### A native APP for the phone/tablet
#### A website with a SPA dashboard and an _expert_ blog about onirism 
#### An API to handle all requests and responses

#### Native APP has a 3-mode behavior
- User and wearables setup
- Sentinel mode
- Oracle mode

##### Setup
Pairing devices
Personal data and preferences for
- Sleeping hours
- Devices sensitivity 
- Disturbance mode
- .../...

##### Sentinel mode
Must be activated explicitly. In short, watches glasses activity and logs all significant activity. Those logs will be upload later.
 
- APP device enters watching mode and sends start signals to wearables
- APP inits a new session to API with a small payload 
- API returns a token to identify the session 
- APP watches glasses activity and analyzes the likeliness of a positive match
- Sends a disturbance signal to disturbance device if input is positive
- Logs date and time of positive and negative inputs 
- Sends a system notification linked to signal for user to activate APP when awake 
- Sends all log stored during session along with an end of session payload
 
##### Oracle mode
The APP shows the activity for past periods
Asks few questions about sleeping experience:

- Did you sleep well?
- Do you remember dreaming?
- Wizard mode/smart-path algorithm:
	- Did you appear in dream? Yes / No
	- Where you alone or with somebody else? Alone / With a relative / With unknown people 
	- Dream happened in a known location? Yes / Somewhat yes / In a really strange place
       - .../...
- Wizard mode helps build a different profile for each user
- Path can contain adult content questions
- Questions on wizard mode can ask API for new branches on questions path
Those questions are, in fact, a reinforcement of dream memories if any. Helps user remember his dream creating a boost effect, so the product seems more effective. 


#### WEB Dashboard
- Allows user to login and analyze data produced
- Allows user to modify preferences data not tied to wearables
- Allows user to review past dream data
- Allows user to create a past dream experience using a smart-path decision tree as in the APP


#### API
API has three different endpoint families for three different purposes

- Auth/User endpoint. With recovering options, to manage all user related data and activity.
- LOG  & device endpoints. To initiate and close sessions and log all data created by the sentinel APP. Also checks devices profiles and updates.
- Oracle endpoints.To Fetch and store data related to smart-path algorithm



## System Architecture
#### Data flows

##### Device calibration and registration

|APP  |   | DEVICE   |   |API   |
|---|---|---|---|---|
| Scans nearby devices  | ->  | HELO  |   |   |
| |<- | Pairing  | | |
| Name device| -> |  | | |
| | <- | ACK | | |
| Register device|  |  | -> | Validation|
| |  |  | <- | Sends ID's|


##### Data harvest. Sleep mode

|DEVICE  |   | APP   |   |API   |
|---|---|---|---|---|
| | | User opens APP  | | |
| | | Ask API for session token w/payoad | -> | |
| | | | <- | Sends back session ID|
| | <- | Asks device | | |
| Data| -> | | | |
| | <- | Filter signal. Stores data| | |
| Data| -> | WOW signal. | | |
| | <- | Send disturbance data | | |
| ACK | -> | Stores data | | |
| | | Creates notification | | |

##### Data upload

|APP  |   |API   |
|---|---|---|
| User activates notification. LOG uploaded   | ->  | Stores  |
|   |<- |OK  |
| User completes smart-path. Sends END payload   | ->  | Stores  |
|   |<- |OK  |

##### Dream upload

|APP  |   |API   |
|---|---|---|
| User request dream annotation   | ->  |   |
|   |<- | Sends first question with 3 answers  |
| User answers YES/NO/DON'T KNOW   | ->  |   |
|   |<- |Sends following questions with X answers  |
|   | repeat | |
|  | <- |  Sends End of story|


#### Schemas
##### AUTH & USER API Architecture
![](aut_user_api.png) 


##### LOG & DEVICE API
![](log_api.png) 

##### ORACLE API & BI CONNECTION
![](oracle_bi.png) 



## Tools & technologies
#### Mock devices. Simulation of wearables
- Python powered using bluetooth libraries

#### Mobile APP
- iOS. Standard Swift/ObjC API's
- Android. Android > 4.0
- No need of C programming on both platforms

#### Dashboard
- SPA Application. React / ELM
- Javascript toolchain. Webpack


#### API
##### Auth API.
- GO backend
- Provides auth platform across all others API's
- REST interface. GO binary interface
- Storage: Mongo DB. (DynamoDB on AWS or BigTable on GC)
    - Provides schemaless models, ideal for managing users from different resources
    - Document versioning capable
    - Scales well both vertically & horizontally

##### Events & devices API.
- GO backend
- Storage: Cassandra (RDS on AWS or SQL on GC)
    - Timeseries LOG type storage
    - Append only backend. No need to change data
    - Reading optimized by cassandra internal architecture. Fast reads

##### Oracle API.
- Python backend
- Storage: Neo4j
    - Graph database. State + Action -> State
    - Capable of storing logical paths of a plot by using Actors, Scenarios and Actions, and that's what a dream is.

Developer X Activities 
In order to make developers active across several parts of the project a possible developer activity matrix could look like:

|Role  | iOS | Android | Auth API | Events API | Oracle API | WEB | QA | B.I | DevOps |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|iOS dev    |+++| + |   |   |   | + |   | + |++ |
|Android dev| + |+++|   |   |   | + |   | + |++ |
|go dev     |   |   |+++|+++|   |   |   | + |++ |
|Python dev |   |   |   |   |+++|+++|   |++ | + |
|QA dev     |   |   |   |   |   | + |+++|   |   |
|Data Sc.   |   |   |   |   |   |   | + |+++|   |

+++ = Main task | ++  = Could collaborate actively | + = Could lend a hand if necessary


## Testing
##### Software testing
I suggest adopt a BDD approach to cover Integration and E2E tests.
BDD gives the abstraction of natural language to cover all aspects of the application, and since the whole product requires the expertise of quite different fields, a common language used by everybody could help.

[Behave](http://pythonhosted.org/behave/), a BDD python framework is a perfect choice for this, since the definition of scenarios link directly to user stories that should be fulfilled at its turn by different teams.

Each application, also, must provide its complete Unit Tests

##### Hardware testing
A Virtual Device Emulator must be created to test APPs, VDE runs on developer machine as a bluetooth device and is capable of creating same signals as emulated device.

This VDE can be easily created using Python or GO and provides a minimal console to create signals and monitor communication.



## Deploy & monitor
For API development I suggest a docker approach where all applications can be containerized and deployed using docker compose, that will simplify iOS/Android and QA development by creating production-like environments on local machines.

This approach will simplify also deployments by using well tested base docker machines.

Jenkins can use those containers to run integration tests before a deploy, and deploy stack if successful.

  
Since APPs are quite verbose on sending data and states to API, a simple log analysis could reveal many interesting patterns:

- Devices that could not connect or report many errors of pairing/comunication
- Devices that report no REM activity
- Devices that report constant REM activity
- Sleeping sessions that ended with a lucid dream (yay!)

## Development Plan
Product seems perfect for a MVP approach, beginning with a simple minimal setup with no complex API's around.


Version 1.0 could be as minimal as:

- Wearables
- iOS/Android APP to control wearables and store activity locally on the device 

Later all features can be added depending on user needs

- 1.2 Remote API storage for data. Need to have the LOG & devices API
- 1.4 User login + WEB Dashboard. Need to have Auth/User API
- 1.6 Annotate dreams. Need to have Oracle API 
- .../...
- 2.0 Dream interpretation mode. Need some B.I. analysis over collected data.



## Business Model
APP and dashboard services are free to user
Data collected builds a valuable dataset over time, and user profiles beyond administrative data can be easily generated by analyzing logs and dream plots.

Money income comes in three ways as __premium services__, __advertising__ and __product referrals__



#### Premium services.
On oracle mode premium versions gives user access to:
- Adult content questions on wizard path of dream annotation. One-time unlock
- Analysis of dreams. Pack of reports buying mode. 10 reports for 1.99!
- Auto tune sensors. Based on success rates and configurations and profiles of users
- Remove advertising on Dashboard

#### Advertising
Only on dashboard and curated to be related to user profile

#### Product referrals 
Blog should provide information about dreams, sleeping and all related world to the act of dreaming. People interested on having a better dream experience will be a likely follower of the blog hence the visitor profile has a strongly defined set of interests.

Articles can suggest different products to enrich sleep experience, so those products can easily be referred by us generating revenue on each successful transaction.