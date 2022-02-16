# Quick start guide

The purpose of the quick start guide is to provide the simplest scenario to integrate with the Income API.

## Prerequisites

 * Client ID and Client Secret - this will be provided by SprintHive, it is used to create an JWT token which will be required when calling the Income API
 * An environment to test against - this will provisioned by SprintHive
 * An example bank statement

> A postman collection can be found in the postman directory
                                                            
## Process Flow

These are the 3 main API calls which will be made between your system and SprintHive
* Create an income verification - a POST request with a JSON body 
* Upload a bank statement - a Multipart form with a file parameter
* Get the result - a GET request which returns JSON 

![quick-start-guid-sequence-diagram](images/quick-start-sequence-diagram-1.png)
