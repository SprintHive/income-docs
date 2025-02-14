# Income FAQ

## Are our API's static or dynamic?


Our APIs are dynamic and updated through additive changes and ensuring no backward breaking changes.  
Response structures may vary depending on the income verification's state (e.g.retrieving a document - 
different documentType's will result in different response structures).   
Deployments may occur without explicit notification, but they will always maintain backward compatibility.    
Clients should implement robust data parsing logic to handle potential unrecognised fields.
