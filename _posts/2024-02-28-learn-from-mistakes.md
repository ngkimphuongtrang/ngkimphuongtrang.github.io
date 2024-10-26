---
title: "[T] My coding mistakes found on PROD"
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
To avoid repeatedly assigning `response.StatusCode` to `item.StatusCode` in every conditional block, I opted to assign the value once at an earlier point. 
Unfortunately, this change led to unintended consequences. After several weeks of stable operation in production, the server began to experience issues under high load, with some API requests timing out, leading to `net/http: request canceled (Client.Timeout exceeded while awaiting headers)` errors. 
Consequently, the production application encountered a runtime error, causing it to malfunction.
### How I fixed the problem?
Add a check nil object before accessing their value
```go 
if response != nil{
	item.StatusCode = response.StatusCode
}
```

## Insert records with default value created_at, updated_at into database table
(2023-11-15)
### How issue happened?
My corrupted code:
```go 
const sqlCreateOutputRow = `
INSERT INTO X.output_rows(
                                 client_code,
                                 request_id,
                                 status_code,
                                 created_at,
                                 updated_at)
VALUES %s ON DUPLICATE KEY UPDATE encrypted_score = VALUES(encrypted_score), score_request_id = VALUES(score_request_id), error = VALUES(error), status_code = VALUES(status_code);`

const defaultBatchSize = 1000

// OutputRow  holds data for each row that will be written into output file.
type OutputRow struct {
	ID               int64     `json:"id"`
	ClientCode       string    `json:"client_code"`
	RequestID        string    `json:"request_id"`
	StatusCode       int       `json:"status_code"`
	CreatedAt        time.Time `json:"created_at"`
	UpdatedAt        time.Time `json:"updated_at"`
}

func (s *Store) CreateManyOutputRows(ctx context.Context, outputRows []*OutputRow) error {
	var valueStrings []string
	var valueArgs []interface{}

	for _, outputRow := range outputRows {
		valueStrings = append(valueStrings, "(?, ?, ?, ?, ?)")

		valueArgs = append(valueArgs,
			outputRow.ClientCode,
			outputRow.RequestID,
			outputRow.StatusCode,
			outputRow.CreatedAt,
			outputRow.UpdatedAt)
	}

	sql := fmt.Sprintf(sqlCreateOutputRow, strings.Join(valueStrings, ","))
	err := s.DB.Exec(sql, valueArgs...).Error
	if err != nil {
		log.Error(ctx, err)
		return err
	}

	return nil
}
```
Typically, when adding a new data row, I don't manually set the `created_at` and `updated_at` column values since they are automatically populated by the database. 
However, on this occasion, GitHub Copilot generated a code snippet that included these fields, and because I didn't specify their values in the object, they defaulted to their initial values, which caused unexpected behavior in my application.

### How I fixed the problem?
Remove segment of created_at and updated_at in the code
```go 
const sqlCreateOutputRow = `
INSERT INTO X.output_rows(
                                 client_code,
                                 request_id,
                                 status_code)
VALUES %s ON DUPLICATE KEY UPDATE encrypted_score = VALUES(encrypted_score), score_request_id = VALUES(score_request_id), error = VALUES(error), status_code = VALUES(status_code);`

const defaultBatchSize = 1000

// OutputRow  holds data for each row that will be written into output file.
type OutputRow struct {
	ID               int64     `json:"id"`
	ClientCode       string    `json:"client_code"`
	RequestID        string    `json:"request_id"`
	StatusCode       int       `json:"status_code"`
	CreatedAt        time.Time `json:"created_at"`
	UpdatedAt        time.Time `json:"updated_at"`
}

func (s *Store) CreateManyOutputRows(ctx context.Context, outputRows []*OutputRow) error {
	var valueStrings []string
	var valueArgs []interface{}

	for _, outputRow := range outputRows {
		valueStrings = append(valueStrings, "(?, ?, ?)")

		valueArgs = append(valueArgs,
			outputRow.ClientCode,
			outputRow.RequestID,
			outputRow.StatusCode)
	}

	sql := fmt.Sprintf(sqlCreateOutputRow, strings.Join(valueStrings, ","))
	err := s.DB.Exec(sql, valueArgs...).Error
	if err != nil {
		log.Error(ctx, err)
		return err
	}

	return nil
}
```