---
title: "My mistakes found on PROD"
date: 2024-02-28
---

## Access nil object 
(2023-12-18)
### How issue happened?
My corrupted code:
```go 
func (j *Job) Execute(ctx context.Context){
	for index, item := range j.Data{
		response, respData, err := j.sendRequest(j.ctx, item)
		item.StatusCode = response.StatusCode
		if err != nil || (response.StatusCode != http.StatusOK && response.StatusCode != http.StatusNotFound){
			
        } else if response.StatusCode == http.StatusNotFound{
			
        } else{
			
        }               
    }   
}
```
To avoid repeatedly assigning response.StatusCode to item.StatusCode in every conditional block, I opted to assign the value once at an earlier point. Unfortunately, this change led to unintended consequences. After several weeks of stable operation in production, the server began to experience issues under high load, with some API requests timing out, leading to net/http: request canceled (Client.Timeout exceeded while awaiting headers) errors. Consequently, the production application encountered a runtime error, causing it to malfunction.
### How I fixed the problem?
Add a check nil object before accessing their value
```go 
if response != nil{
	item.StatusCode = response.StatusCode
}
```