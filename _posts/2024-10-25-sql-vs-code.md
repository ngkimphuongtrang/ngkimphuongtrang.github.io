---
title: "[G] Finding Emails Appearing in Multiple Countries: SQL vs. Code"
date: 2024-02-26
---

In this post, we will tackle a common data processing task: identifying email addresses that appear in more than one country. We will explore two approaches to solving this problem: using SQL and using code.

## Problem Statement
We have a database table with the following columns:

- person_number
- site_country
- person_email_address

The person_email_address column contains a list of email addresses separated by commas. Our goal is to find email addresses that appear in more than one site_country.

Example
Consider the following data:
```
person_number | site_country | person_email_address
---------------------------------------------------
1             | USA          | example1@domain.com,example2@domain.com
2             | CAN          | example2@domain.com
3             | USA          | example3@domain.com
4             | MEX          | example1@domain.com
```
In this example, example1@domain.com appears in both the USA and MEX, and example2@domain.com appears in both the USA and CAN.

## Alternative SQL Solution
Despite the challenges, it is possible to solve this problem using SQL with the help of common table expressions (CTEs) and string functions. Here is an example approach:

#### Step-by-Step SQL Solution
1. Split the Email Addresses: Use a recursive CTE to split the comma-separated email addresses into individual rows.
2. Map Emails to Countries: Create a mapping of emails to their respective countries.
3. Identify Emails in Multiple Countries: Group by email and count the distinct countries.
```sql
WITH RECURSIVE EmailSplit AS (
    SELECT
    person_number,
    site_country,
    SUBSTRING_INDEX(person_email_address, ',', 1) AS email,
    SUBSTRING_INDEX(person_email_address, ',', -1) AS remaining
    FROM table_name UNION ALL
    SELECT
    person_number,
    site_country,
    SUBSTRING_INDEX(remaining, ',', 1) AS email,
    SUBSTRING_INDEX(remaining, ',', -1) AS remaining
    FROM EmailSplit
    WHERE remaining != email
)
SELECT email, GROUP_CONCAT(DISTINCT site_country) AS countries
FROM EmailSplit
GROUP BY email
HAVING COUNT(DISTINCT site_country) > 1;
```
#### Explanation
1. EmailSplit CTE: This recursive CTE splits the person_email_address column into individual email addresses.
2. Mapping Emails to Countries: The result of the CTE is a mapping of each email to its corresponding site_country.
3. Identifying Emails in Multiple Countries: The final query groups by email and counts the distinct countries. Emails appearing in more than one country are selected using the HAVING clause.

## Code Solution
To overcome the limitations of SQL, we can download the data into a CSV file and process it using Go. Here is the complete Go code for the solution:

### Step-by-Step Solution
1. Read the CSV File: Use the encoding/csv package to read the CSV file.
2. Process Each Record: Split the email addresses by commas and map them to their respective countries.
3. Identify Emails in Multiple Countries: Check the map to find email addresses that appear in more than one country.
```go
package main

import (
"encoding/csv"
"fmt"
"os"
"strings"
)

func main() {
// Open the CSV file
file, err := os.Open("data.csv")
if err != nil {
fmt.Println("Error opening file:", err)
return
}
defer file.Close()

	// Read the CSV file
	reader := csv.NewReader(file)
	records, err := reader.ReadAll()
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	// Create a map to store email to countries mapping
	emailToCountries := make(map[string]map[string]struct{})

	// Iterate over the records
	for _, record := range records {
		personNumber := record[0]
		siteCountry := record[1]
		personEmailAddresses := record[2]

		// Split the emails by comma
		emails := strings.Split(personEmailAddresses, ",")

		// Iterate over the emails and populate the map
		for _, email := range emails {
			if emailToCountries[email] == nil {
				emailToCountries[email] = make(map[string]struct{})
			}
			emailToCountries[email][siteCountry] = struct{}{}
		}
	}

	// Find emails that appear in more than one site country
	for email, countries := range emailToCountries {
		if len(countries) > 1 {
			fmt.Printf("Email: %s appears in countries: ", email)
			for country := range countries {
				fmt.Printf("%s ", country)
			}
			fmt.Println()
		}
	}
}
```

#### Explanation
1. Reading the CSV File: The file is opened and read using the encoding/csv package. We handle any errors that may occur during this process.
2. Mapping Emails to Countries: We create a map emailToCountries where the key is the email address and the value is another map (used as a set) to store the countries.
3. Processing Each Record: We split the email addresses by commas and populate the emailToCountries map with each email and its corresponding country.
4. Finding Emails in Multiple Countries: Finally, we iterate over the map to find and print email addresses that appear in more than one site_country.

## Analysis: SQL vs. Code
### SQL Approach
#### Advantages:

- Direct Database Query: The SQL approach allows you to work directly within the database, eliminating the need for data export and import.
Reusable Query: The query can be easily reused and adapted for similar tasks.
#### Disadvantages:

- Complexity: Handling complex string manipulations and recursive CTEs can be challenging and may not be supported by all SQL databases.
- Performance: Recursive queries and string operations can be slow, especially on large datasets.
### Code Approach
#### Advantages:

- Flexibility: Go provides powerful string manipulation functions and data structures that make it easy to process and analyze data.
- Performance: Go's performance is generally better for complex data processing tasks compared to SQL.
#### Disadvantages:

- Data Export/Import: You need to export data from the database to a CSV file and then import it into the Go program, which adds an extra step.

## Personal Observations
To me, I first thought of the code solution because it is easy to run code on my local machine compared to running a complex query on the database system. I also worried that the complex query might result in a timeout. Therefore, I used the code approach to solve my problem first.

While writing this analysis, I decided to try the SQL solution, and it worked actually. With my data table containing 875,165 rows, the result had 143 rows, and the query finished in 46 seconds:

```
{code: 200, errors: null, message: null, processTime: 46.910745927,…}
```
## Conclusion
Both SQL and code have their strengths and weaknesses. The choice of approach depends on your specific requirements and constraints. If you need to work directly within the database and prefer a reusable query, the SQL approach may be suitable. However, if you require more flexibility and performance for complex data processing tasks, the code approach is a better choice.

Feel free to try these solutions with your own data and modify them according to your specific requirements. Happy coding!

