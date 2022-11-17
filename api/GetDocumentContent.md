# Downloading a document

The following endpoint can be used to download the file from SprintHive and then upload it to your systems file storage.

Endpoint: ```/document-management/v2/admin/document/${documentId}/content```  
Method: GET
Response body: The byte array for document
Response header: The file name is returned using key content-disposition in the response header     
