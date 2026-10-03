# Agentforce-Proactive-Service

Create an Ligthning app on Salesforce Developer Agentforce org.


Add Standard objects: Account, Opportunity, Cases, Report, Dashboard and Create Custom objects: Device, Device Telemetry, Agent Action Logs. 


Lookup Relationship(detail->master): Opportunity->Device, Case->Device, Agent Action Log->Device, Customer(field on Device object)-> Account


Master Detail Relationship: Device Telemetry->Device


Record Trigggered Flow 1:
Case Increment Critical Failure count.
Step1. Object- Case, when record is created. All Conditions Are Met (AND): Risk Level= High, Device Is Null False. Actions and Related Records.

Step2: Add Get Records(Get Parent Device). Object- Device. Condition: Id= $Record.Device__c; Store only the first record, Automatically store all fields.

Step3: Add Assignment(Increment Failure Count). Total_Critical_Failures__c Add 1.

Step4: Update Records(Update Device Record). Use the IDs and all field values from a record or record collection. Record or Record Collection: Device from Get_Parent_Device.


Record Trigggered Flow 2:
Device Create Replacement Opportunity.

Step1: Object- Device, when a record is updated, All Conditions Are Met(AND):Total_Critical_Failures__c = 3.

Step2: Create Records(Create Replacement Opportunity). object- opportunity. set values:
name-{!Opportunity_Name_Formula}, stage-prospecting, closedate {!$Flow.CurrentDate} + 30, AccountnId {!$Record.Customer__c}, Device {!$Record.Id}.


Record Trigggered Flow 3:
Case - Device Upgrade Engine
Step1: Object-Case, when a record is created. Device is Null False. Actions and related records.

Step2. Get Records(Get Historical Cases). Object- case. Device = {!$Record.Device__c}. Store All Records, Automatically store all fields.

Step3. Add Assignment(Count Cases). varCaseCount = {!Get_Historical_Cases}.

Step4. Add Decision(Has it failed 3 times?)

Step5. When Yes-3 or more,

Create records(Create Replacement Opportunity). Object -Opportunity. Field values:
Account ID- {!$Record.Device__r.Customer__c}, Close Date- {!CloseDate30Days}, Name- {!$Record.Device__r.Name}, Stage- Prospecting.

Step6. Default value when Under 3 failures.


PROMPT:		
New Prompt Template- Flex- Template Name: IoT Risk Analyzer- Define the inputs: name(TelemetryRecord, DeviceRecord)- Write the Prompt.


Record Trigger Flow 4 : 
AI Risk Orchestrator

Step1: Object- Device Telemetry, when record is created.Custom Condition Logic is Met. Condition Logic 1 OR 2.
1.Temperature>140
2.Usage Hours>6
Actions and related records.

Step2: Add Get Records (Get Parent Device). Object-Device, All Conditions are met(AND) Record ID = {!$Record.Device__c}. Only the first ecord to store, Automatically store all fields.

Step3: Add action(Run AI Analysis)
RecordDevice- {!Get_Parent_Device}
TelemetryRecord- {!$Record}.

Step4. Add Decision(Is Risk High?)
When High Risk:
1. Add  Decision(Is Autonomous Mode On?) 
When Autonomous ON
    -> Create Records(High Priority Case). Objcect- Case. Set field values: Device-{!$Record.Device__c}, Prediction Explanation-{!Run_AI_Analysis.promptResponse}, Priority-High, Risk Level-High.
    ->Create Records(Log Autonomous Case). Object- Agent Action Log. field values: 
    Action Type- Case Created, Device- {!$Record.Device__c}, Reason- {!Run_AI_Analysis.promptResponse}
   
2. When Manual(Log Only)
    ->Create Records(Log Manual Recommendation). Object- Agent Action Log. field values: 
Set field values: Action Type- Action Skipped (Manual Mode), Device- {!$Record.Device__c}, Reason- {!Run_AI_Analysis.promptResponse}.

Step5. Default value when No High Risk.


Write an apex class for schedule: TelemetrySimulator.(Flow instantly caught the data, sent it to Agentforce, and—if the random temperature was over 140—the AI created a Case!)


Automate the code: quick box > apex class > Schedule apex > job name: Daily IoT Simulation. > select apex class(TelemetrySimulator) > weekly > check everyday.


Create Report 1 (Prevented Incidents):This report proves that your AI is successfully generating predictive cases.
Report2(AI Decisions): This report audits the background work your Agentforce AI is doing.


Create Dashboard (Proactive AI Command Center): This wrapped all in a professional UI.

