# MongoDB Atlas + Microsoft PowerApps/PowerAutomate

## Description

   #### This use-case shows how easily a front end for the Customer Onboarding can be developed using Power Apps and the back end process of validating the uploaded proofs can be automated and triggered in real-time using Power Automate and this entire application can seamlessly integrate with MongoDB Atlas which stores the details entered by user on the front-end and also stores the results of the validation.

#### Below diagram depicts the workflow  and the components involved in the solution:

<img width="700" alt="Screenshot 2022-10-20 at 9 20 47 PM" src="https://user-images.githubusercontent.com/101181433/196997370-51d3fe0f-55bf-432c-a106-de570eba5cd9.png">

  #### 1: User enters the details and submits the Onboarding Form .
  #### 2: Entered information is inserted into MongoDB collection and Proof document uploaded in the form is stored in Azure Blob Storage and this will trigger a Power Automate Workflow.
  #### 3: Power Automate workflow will scan the uploaded proof document 
  #### 4: Scanned document is validated with the details present in the MongoDB collection.
  #### 5: Validation status is emailed to the user (Yes/No).
  #### 6: MongoDB document is updated with the validation status. 

## Prerequisites

  #### 1: Spin up a MongoDB Atlas cluster [Click here](https://account.mongodb.com/account/login)
  #### 2: Azure subscription to store the files uploaded in Blob storage [Click here](https://azure.microsoft.com/en-in/free/)
  #### 3: Power Apps/Power Automate License [Click here](https://powerapps.microsoft.com/en-us/pricing/)

## Below are the High level steps followed for the setup:

#### 1: [MongoDB Cluster Setup](#mongodb-cluster-setup)
#### 2: [Create a Blob Storage](#create-a-blob-storage)
#### 3: [Connect to MongoDB Custom connector](#connect-to-mongodb-custom-connector)
#### 4: [Build Power Apps Onboarding form](#build-power-apps-onboarding-form)
#### 5: [Build Power Automate Flow](#build-power-automate-flow)

# Setup
## MongoDB cluster Setup

### A: Configure Atlas Environment

  - Log-on to Atlas account.
  
  - In the project's Security tab, choose to add a new user called main_user, for this user select Add Default Privileges and in the Default Privileges section add the roles readWriteAnyDatabase.
  - Create a Sandbox Cluster in a cloud provider region of your choice with default settings.
  - In the Security tab, add a new IP Whitelist for your laptop's current IP address
### B: Enable the Data API in Atlas
  - In the left-hand Deployment menu, click to navigate to Data API.

  - Slide the Data API Enabled toggle to ON for the cluster you're using for this PoV. Select Enable when prompted and wait for the deployment to complete:

   <img width="600" alt="DataAPI" src="https://user-images.githubusercontent.com/101181433/185349961-ac6fb7eb-c78e-4bda-b6ba-3d10adfb6ac6.png">

  - Copy your Data API App ID from the URL Endpoint as you will need it for Postman. In the above screenshot, data-qdcie is my App ID.
### C: Generate an API key
    
   - Switch to the API Keys tab and click Generate API Key.
   - Give your API key an appropriate name then click the Generate API Key button.
   - Copy the private API Key for use and safekeeping that is displayed once generated. 
   ### Note: This is the only time you can retrieve the full private key.

## Create a Blob Storage

  - You can follow the steps provided in [Microsoft Link](https://azure.microsoft.com/en-in/free/).
  - Create 2 containers "documents" and "filestoprocess"
     - documents -> When the user clicks on Upload and uploads a file, it is stored here. This does not trigger the Power Automate Flow
     - filestoprocess -> When the user clicks on Submit, file is copied from documents to filestoprocess container and this in-turn will trigger the Power Automate Flow

## Connect to MongoDB Custom connector
  - Login to Power Apps.
  - Follow below 3 steps from screenshot to use the MongoDB Custom connector.
  
<img width="150" alt="Step1" src="https://user-images.githubusercontent.com/101181433/196114084-020ec9e7-183b-4a03-9900-a0278e1b1a9a.png">
<img width="200" alt="Step2" src="https://user-images.githubusercontent.com/101181433/196098527-3a8cec52-7767-4ff8-8f46-8f42c2b28ba5.png">
<img width="300" alt="Step3" src="https://user-images.githubusercontent.com/101181433/196664796-9299fc43-0eeb-4dbc-b118-920d635ef578.png">


  - Change the name of the connector as per your choice
  - Toggle the “Swagger Editor” button which will show the swagger code for the connector. Replace the swagger file content with the MongoDBDataAPI.swagger.
      
   - Update the connector.
   - Go to Test Tab → New Connection. Enter the private API Key created in Generate an API key step.
   
 <img width="400" alt="testTab" src="https://user-images.githubusercontent.com/101181433/185431420-4ee47fbf-193d-40dd-8401-cfbe8e9ef61e.png">

   - This connection created will show up in the Dataverse→Connections in the Left Panel of Power Apps portal.
   
## Build Power Apps Onboarding form
  - Go to Apps in the left panel or Power Apps Portal.
  - Create a new Canvas app. Give a name, choose Tablet Mode.
   <img width="300" alt="TabletMode" src="https://user-images.githubusercontent.com/101181433/185431906-9bca0b70-4e25-4bd5-8d62-89ea4e426b6a.png">

  - ## Add data source 
    - ### Blob Storage
      - Type Blob Storage → Add a connection. Enter the Azure storage account Name and Azure storage account Access Key(Get it from the Azure Portal where the storage container is created). Click Connect.
      
        <img width="209" alt="DataSource" src="https://user-images.githubusercontent.com/101181433/185431920-9a508684-c919-42d9-8de6-dcd83dcd2b65.png">

    - ### MongoDB DataAPI
      - Type MongoDB and you will see the connection you created with the Custom Connector.

  - ## Create the Forms
    - We need 7 screens 
     ### 1. Welcome Screen: 
     
     User can Open a New Application or choose to Open an Existing Application saved earlier by providing the Application Number.
      
     <img width="300" alt="Welcome" src="https://user-images.githubusercontent.com/101181433/196109017-9c07440e-f021-40c4-bf55-871fb37e2064.png">
         
     - Create 2 Buttons by clicking on Insert-> Input-> Button.
     - Click on the Button:
         
         - Change the name in the Text Property for both the buttons.
         - Choose the OnSelect property of "Open New Application" and add this function:
            
                  Set(ApplNo,Text(RandBetween(1,9999)));
                  Navigate('New Form',ScreenTransition.Fade,{ApplnoNew:ApplNo});
                     
         - Choose the OnSelect property of "Retrieve Existing Application" and add this function:
            
                  Navigate(GetApplicationNumber,ScreenTransition.Fade);
         
     ### 2. New Application: 
     
     Clicking on New Application takes us to the below form to fill all the details.
         
     <img width="350" alt="New Form" src="https://user-images.githubusercontent.com/101181433/196110393-8adc8484-fda6-48e6-8107-619bd9987569.png">
         
     - Create a New Screen.
        - Heading and labels on the left. 
           - Click on Insert → Label and key in the name.
        - Input boxes on the right, 
           - Click on Insert → Text → Text Input.
           - Click on the first Text Input box, go to properties and add “Please enter firstName as mentioned in your passport” in the Hint text property. Similarly add Hint text to each of the Text Inputs.
           - For the Application Number, we need to show the random number generated in the Welcome Page. So add a label set the Text as "ApplnoNew"(which is the variable we are passing from the Welcome Screen)
            - Rename the Labels,TextInputs and Buttons as shown in the below screenshot. This will be easier to identify the fields for the actions/functions we write for Buttons.
            
               <img width="200" alt="leftPanel" src="https://user-images.githubusercontent.com/101181433/196669611-cd2cd6b7-e983-423b-8bc2-a236e420b1a1.png">

           
         - ### Upload Button:
         
            - Click on Insert → Media → Add Picture. Edit the text to “Upload”.
            - Click on the Media Button created→ choose OnSelect Option → Add this function: 
            
                  Set(varAzureFile,AzureBlobStorage.CreateFile("documents",
                  Concatenate(Passport.Text,".pdf"),Upload.Media));
                  Collect(collectTemp,{FN:Upload.FileName,FL:"[Url of your blob storage container 
                  Eg: https://mystorageaccountname.blob.core.windows.net]" & varAzureFile.Path,FID:varAzureFile.Id});
                    
                 - Filestoprocess →  Folder name in Blob Storage
                 - Concatenate(Passport.Text,".pdf") →  File name( I chose it to be the Passport number entered by the user in the TextInput we created earlier)
                 - Upload.Media → For Uploading the file.

          - ### Add a gallery to show the uploaded file:
          
             - Click on Gallery and select vertical
             - Choose the CollectTemp as the datasource(CollectTemp is the collection created during the Upload).
             - For the title of the Gallery. Select Text and set it to
             
                     ThisItem.FN
                     
             - Add the delete button from Insert-> Icons. Click on the Button created→ choose OnSelect Option → Add this function: 
                 
                      Remove(collectTemp,ThisItem);
                      AzureBlobStorage.DeleteFile(ThisItem.FID);
                      Reset(Upload);
                      
               <img width="200" alt="gallery" src="https://user-images.githubusercontent.com/101181433/196670006-36772b75-5c60-45bf-9a7c-409909d892bf.png">
                 
          - ### Save Button:
            - Click on Insert → Button. Edit the text to “Save”.
            - Click on the Button created→ choose OnSelect Option → Add this function: 
            
                     MongoDB.InsertDocument("Content-Type","Access-Control-Request-headers","api-key",
                     {dataSource:"Sandbox",database:"XYZBank",collection:"onboarding",document{firstname:Upper(Fname.Text),
                     lastname:Upper(Lname.Text),DateOfBirth:DOB.Text,passportNumber:Passport.Text,_id:Passport.Text,
                     emailId:email.Text,applicationNumber:ApplNoLabel}});
                     
                     Navigate('Save success',ScreenTransition.Fade,{returnAppNumber:ApplNoLabel});

          - ### Submit Button:
            - Click on Insert → Button. Edit the text to “Submit”.
            - Click on the Button created→ choose OnSelect Option → Add this function: 

                    If(IsBlank(Fname.Text) Or IsBlank(Lname.Text) Or IsBlank(DOB.Text) Or IsBlank(Passport.Text) 
                    Or IsBlank(email.Text) Or IsEmpty(collectTemp.FN),Set(popup,true),
                    UpdateContext({_deleteFile:AzureBlobStorage.GetFileMetadataByPathV2("csg10032001f02ad0eb",
                    Concatenate("documents/",Passport.Text,".pdf")).Id});
                   
                    AzureBlobStorage.CopyFile(Concatenate("documents/",Passport.Text,".pdf"),
                    Concatenate("filestoprocess/",Passport.Text,".pdf"),{overwrite:true});
                    
                    AzureBlobStorage.DeleteFileV2("csg10032001f02ad0eb",_deleteFile);

                    MongoDB.InsertDocument("Content-Type","Access-Control-Request-headers","api-key",      
                    {dataSource:"Sandbox",database:"XYZBank",collection:"onboarding",document:
                    {firstname:Upper(Fname.Text),lastname:Upper(Lname.Text),DateOfBirth:DOB.Text,passportNumber:Passport.Text,
                    _id:Passport.Text,emailId:email.Text,applicationNumber:ApplNoLabel.Text}});

                    Navigate(Success,ScreenTransition.Fade,{returnAppNumber:ApplNoLabel});
                    );

         - ### Reset Button:
           - Click on Insert → Button. Edit the text to “Submit”.
            - Click on the Button created→ choose OnSelect Option → Add this function:

                  Reset(Fname);Reset(Lname);Reset(DOB);Reset(Passport);Reset(email);
                  
          - ### Popup Window:
              -  We also need a popup window to stop the user if he is trying to submit without entering all the details.
              - Click the form(anywere on the screen)-> Under the Advanced-> Action -> OnVisible, Add 
                  
                     Set(popup,false)
                     
              - Click on the '+' Button on the left panel. Add a Rectangle. Adjust the size, color, border.
              - Inside the Rectangle add a Label and add the below line under text Property :
              
                  "Please fill in all details and Upload your passport as a pdf before clicking on Submit"
              - Inside the Rectangle add a button. Click on the Button created→ choose OnSelect Option → Add this function:
              
                     Set(popup,false)
                     
              - Choose these 3(Rectangle+Label+Button) on the left panel, right click and group them
              
     ### 3. Save Screen:
     
       Saving the form should display us saved successfully message on screen.
         
       <img width="350" alt="Save Success" src="https://user-images.githubusercontent.com/101181433/196110685-ff4aae02-5102-4047-8b7f-d8da189559af.png">
         
       - Create a New Screen.
         - Add a Label and enter the text as: “Your details are successfully saved".
         - Add the Check Icon and Cancel Icon from Icons dropdown as shown in the below screenshot. When the Cancel button is clicked, we need to reset the fields entered in the Welcome Screen and navigate back to the Welcome Screen.
            
           <img width="200" alt="closeButton" src="https://user-images.githubusercontent.com/101181433/196126608-f58db919-b9fa-43ee-b289-2f11c252871c.png">


            - Click on the Cancel Icon created→ choose OnSelect Option → Add this function:

                      Reset(Fname);Reset(Lname);Reset(DOB);Reset(Passport);Reset(email);
                      Navigate('Welcome Screen',ScreenTransition.Fade);

     
     ### 4. Submit Screen:
     
       Submitting the form should display us success message on screen.
       Create a New Screen similar to the Save Screen and change the text message accordingly.
         
        
       <img width="350" alt="Submit success" src="https://user-images.githubusercontent.com/101181433/196110682-81401e95-84c5-4939-aa76-f53354fee932.png">
         
     
     ### 5. Fetch Existing Application:
     If user chooses to open the existing form, it should ask the user to enter the Application Number.
      
     <img width="300" alt="Enter Application Number" src="https://user-images.githubusercontent.com/101181433/196111704-e2e9e42b-b9cf-43fe-a362-d0fb1a1e3d76.png">
         
     - Create a New Screen.
     - Add a label and text field as shown in the screenshot. Close Button should navigate to the Welcome Screen just like we did for previous screens.
     - Add a Submit button. Click on the Button created→ choose OnSelect Option → Add this function:  
                  
                  If(Or(IsBlank(ApplNoEntered.Text),!IsNumeric(ApplNoEntered.Text)),Set(popup,true),

                  UpdateContext({varFormData:MongoDB.FindDocument("Content-Type","Access-Control-Request-headers","api-key",
                  "Accept",{dataSource:"Sandbox",database:"XYZBank",collection:"onboarding",
                  filter:{_id:Blank(),applicationNumber:ApplNoEntered.Text}})});


                  If(IsBlank(varFormData.document.applicationNumber), 
                  Set(popup,true),
                  If(Or(varFormData.document.status= "Valid",varFormData.document.status= "Invalid"),
                  Navigate('View Submitted Form',ScreenTransition.Fade,{varFormData:varFormData}),
                  Navigate('Edit Form',ScreenTransition.Fade,{varFormData:varFormData})

                  );
                  );
                  );
     
     ### 6. Edit Application:
     If the application was saved earlier, then it should take the user to the Edit form page where the entries can be edited.
      
     <img width="300" alt="Edit Form" src="https://user-images.githubusercontent.com/101181433/196112230-ab25fa11-467e-4fba-8fb7-f5874a26219d.png">
         
     - Create a New Screen.
     - Use the similar setup like the New Application. Only changes are with the Save And Submit Button code as these should do a update to the already saved document. Add the Popup window as we did in the New Application.
     - Application number is the number the use enters. We are passing the varFormData collection from the previous page which has all the values retrieved from Database. So for every text Input and the Application Number, Add this under the Default property of the respective fields:
           
                  varFormData.document.firstname
                  varFormData.document.lastname
                  varFormData.document.DateOfBirth
                  varFormData.document.passportNumber
                  varFormData.document.emailId
                  varFormData.document.applicationNumber
                  
     - Click on the Save Button created→ choose OnSelect Option → Add this function:
           
                  MongoDB.UpdateDocument("Content-Type","Access-Control-Request-headers","api-key",
                  {dataSource:"Sandbox",database:"XYZBank",collection:"onboarding",filter:
                  {applicationNumber:ApplNoLabel_1.Text,passportNumber:Blank()},update:{'$set':
                  {firstname:Upper(Fname_1.Text),lastname:Upper(Lname_1.Text),DateOfBirth:DOB_1.Text,
                  passportNumber:Passport_1.Text,emailId:email_1.Text,status:Blank()}}});

                  Navigate('Save success',ScreenTransition.Fade,{returnAppNumber:ApplNoLabel_1});

  
     - Click on the Submit Button created→ choose OnSelect Option → Add this function:  
           
                  If(IsBlank(Fname_1.Text) Or IsBlank(Lname_1.Text) Or IsBlank(DOB_1.Text) Or IsBlank(Passport_1.Text) Or IsBlank(email_1.Text) Or                           IsEmpty(collectTemp.FN),
                  Set(popup,true),
                  UpdateContext({_deleteFile:AzureBlobStorage.GetFileMetadataByPathV2("csg10032001f02ad0eb",
                  Concatenate("documents/",Passport_1.Text,".pdf")).Id});

                  AzureBlobStorage.CopyFile(Concatenate("documents/",Passport_1.Text,".pdf"),Concatenate("filestoprocess/",Passport_1.Text,".pdf"));
                  AzureBlobStorage.DeleteFileV2("csg10032001f02ad0eb",_deleteFile);

                  MongoDB.UpdateDocument("Content-Type","Access-Control-Request-headers","api-key",{dataSource:"Sandbox",
                  database:"XYZBank",collection:"onboarding",filter:{applicationNumber:ApplNoLabel_1.Text,
                  passportNumber:Blank()},update:{'$set'{firstname:Upper(Fname_1.Text),lastname:Upper(Lname_1.Text),DateOfBirth:DOB_1.Text,
                  passportNumber:Passport_1.Text,emailId:email_1.Text,status:Blank()}}});

                  Navigate(Success,ScreenTransition.Fade,{returnAppNumber:ApplNoLabel_1});
                  );


     ### 7. View Submitted Application:    
     If the application was already submitten, then it should be non editable. 
      
     <img width="300" alt="Already Submiited Form" src="https://user-images.githubusercontent.com/101181433/196111714-33a96843-d23b-408a-b56c-1526733e9f31.png">
         
     - Create a New Screen.
     - Use the similar setup like the Edit Application Screen displaying all the retrieved values from varFormData. This screen should be non editable, so for the text inputs-> Properties->Display Mode-> Select View
     - Add a new Label and text field for Displaying the Status of the Application
                          

## Build Power Automate Flow:
  - Login to Power Automate.
  - Create a New Automated Cloud Flow.
  
    <img width="300" alt="AutomatedCloudFlow" src="https://user-images.githubusercontent.com/101181433/185534239-7d7d8740-e2e9-4a43-a47a-2c74210e9f60.png">
    
  - Give a name and choose flow’s trigger as “When a blob is added or modified”. (Hint : Just type blob in the search). Create the first 2 steps in the workflow as shown below.
    #### Note : Storage account name will be the name of the Storage Account created in the Azure Portal.
    
    <img width="300" alt="Step2" src="https://user-images.githubusercontent.com/101181433/185534234-3f90391a-1f9b-4aae-9625-abc7d98539b5.png">
    
  - Create Step3 as below: Choose Dynamic Content → File Content
     
     <img width="300" alt="Step3" src="https://user-images.githubusercontent.com/101181433/185534228-75e045a8-37ca-4401-b5ce-b2dde785d207.png">
     
  - Step 4 and Step 5:

    - These steps are to split the file name that is uploaded in the Blob Container.
    - Step 4 takes Text: Display Name , Search Text :  ‘.’ 
    - Step 5 takes Text: Display Name,starting position: 0 and Length: Text Position(output of Step4)
    
      <img width="300" alt="Step4,5" src="https://user-images.githubusercontent.com/101181433/185534573-e0899d7e-8e51-4d7c-beab-c6d7dfc50493.png">
      
    - Step 6:
      - Type MongoDB and then choose the DataAPI created and choose Find
      - Document as an Action. Enter all the details as below:
      #### Note : _id will be the output of the previous step(Substring)
        
        <img width="300" alt="PAutomate_findDocument" src="https://user-images.githubusercontent.com/101181433/196105643-9f205ae2-fa76-4611-9de6-41364fb70490.png">
        
    - Step 7:
         - Type Parse JSON.
         - Content : output of the Find Operation.
         - Schema→ Generate from sample
         - Paste the JSON output of the previous step and click Done.For Example:
         
                                          {
                                           "document": {
                                             "_id": "N123456",
                                             "firstname": "JOHN",
                                             "lastname": "SMITH",
                                             "DateOfBirth": "2022-01-01",
                                             "passportNumber": "N123456",
                                             "emailId": "xxxxx@xxx.com"
                                                       } 
                                           }
                                           
             <img width="300" alt="Step7" src="https://user-images.githubusercontent.com/101181433/185534557-09ff018b-d8e2-4661-9a11-fb743d2df84b.png">
                                        
     - Step 8:
        - Type Condition.Choose a Value -> Add each field of the Parse JSON output and compare with the AI Model output(Extract information from Identity Documents)
        
            <img width="250" alt="Step8" src="https://user-images.githubusercontent.com/101181433/185534838-c1b96dbc-81b7-4eda-bf77-749d8b92bbad.png">
        
        - If Yes/No:
          - Add an Action → Type Gmail →Send email.
          - Add mail content as per your choice:
          
            <img width="300" alt="Condition" src="https://user-images.githubusercontent.com/101181433/185534832-46ce904e-10e1-4925-aaab-0ab96a0ac951.png">
          
          - The status of the application has to stored back in MongoDB. Under each Action (Yes/No) add a "update document" after the "Send email" Action.For YES status should be "Valid" and for NO status should be "Invalid".
          
          
            <img width="300" alt="PAutomate_UpdateStatus" src="https://user-images.githubusercontent.com/101181433/196105022-a73cee64-f9cd-46df-b127-30d5a5d74c54.png">
         
        This is the final look of Power Automate Workflow:
        
        <img width="250" alt="Final" src="https://user-images.githubusercontent.com/101181433/185534822-c974d9bc-e8e5-453c-a343-a6c5f0721f44.png">
        
# Execution
  1. Click on the Application created in Power Apps.
  2. Enter the User Details in the form.
  3. Click on Save, wait for the Success Message.
  4. Click on Submit, It will popup a message saying "Please fill in all details and Upload your passport as a pdf before clicking on Submit". This is because all the fields and the document is a mandatory before the form is submitted.
  5. Upload the Passport document.Go to Azure Blob Storage, documents container must have the file that is uploaded through the Power Apps.
  6. Click on Submit, wait for the Success Message.
  7. Go to the Atlas collection, there should be an entry made with the details entered in the form.
  8. Go to Azure Blob Storage, filestoprocess container must have the file copied from documents and also the file in documents container would be deleted by now.
  9. Go to Power Automate, Wait for the trigger to get started.(usually takes 1-2 minutes to start the flow).
  10. Wait for the flow to complete. If the form details matches with the uploaded document.You should receive a mail saying Validation is Successful. Else you should receive a Validation failure email.




For any queries, Please contact partners@mongodb.com








