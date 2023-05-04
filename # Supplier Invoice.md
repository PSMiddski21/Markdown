# Supplier Requests 

## Supporting information 
* The following high level sequence diagram shows the services and API calls required for intraction between the messaging backbone and Exchequer when sales invloices or credit notes are sent by the customer  

```mermaid
sequenceDiagram
autonumber
  
  actor U as Buyer
  participant MP as Marketplace
  participant MB as Messaging Backbone
  participant EX as Exchequer
  
  
  participant DTB as DataBay
  participant EXD as ExcheguerDB
  participant RE as Reporting 
  participant UP as UpdateService
  


  U->>MP: Send Creditnote / Invoice (FEE,SIN,PIN)
  MP->>MB: Event Send Invoice
  loop ETL
    MB->>MB: Gather, transform & Load
  end
  Note right of MB: Invoice Transformation 
  MB->>EX: Tranfer Invoice (SFTP)
  loop delivery retry service
    EX->>MB: Deliver failure
    MB->>EX: If failure retry 
    EX->>MB: Confirm delivery success
  end
  MB->>MP: Invoice Transfered 
  loop Gather report data
    DTB->>EXD: Poll invoices 
    note over EXD: Query DB for invoice status
    DTB->>EXD: ExcheguerDB polled
    note over DTB: Polled EXCH Data (6hours)
    note over DTB: Store Polled Data
    RE->>DTB: Gather Report Data
    note over RE: Cache Data
    note over RE: Generate Report
  end
  RE->>MP: Dispaly Report
  U->>MP: View Report
  loop Status check
    UP->>DTB: Check for Status chages
    note over UP: IF Staus Change
    UP->>MB: Trigger Update 
    MB->>MP: Update Status
    end
  U->>MP: View Staus 
