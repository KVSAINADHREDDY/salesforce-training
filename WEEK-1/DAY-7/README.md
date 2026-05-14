# Day 7: Triggers and SOQL

## 1. What is SOQL?
SOQL (Salesforce Object Query Language) is used to read information stored in your org's Salesforce database. It is similar to the `SELECT` statement in SQL. You use SOQL to retrieve specific records that meet given criteria, allowing you to fetch data from standard or custom objects, and filter or sort the results.

## 2. What is an Apex Trigger?
An Apex Trigger is a piece of code that executes before or after specific data manipulation events occur on records in Salesforce, such as insertions, updates, or deletions. Triggers are used to perform custom operations that can't be accomplished with point-and-click tools like Flows, especially when dealing with complex business logic or processing large volumes of data.

## 3. Differences

### Flow vs. Trigger
- **Flow**: A declarative (point-and-click) automation tool used to automate business processes without writing code. Ideal for most standard automation tasks, creating UI wizards (Screen Flows), and simpler logic.
- **Trigger**: A programmatic tool written in Apex code. Used for complex logic, processing large data volumes, recursive operations, or when declarative tools reach their limits. Requires developer skills to write and maintain.

### Before vs. After Trigger
- **Before Trigger**: Executes *before* the record is saved to the database. Used to update or validate fields on the record that is triggering the event itself.
- **After Trigger**: Executes *after* the record is saved to the database (and an ID has been generated). Used to access system-generated field values or to affect changes in other, related records.

## 4. Trigger Use Cases (5 Examples)
1. **Prevent Deletion of Accounts with Opportunities**: An active trigger that prevents a user from deleting an Account if it has related active Opportunities.
2. **Auto-Assign Tasks on Lead Conversion**: When a Lead is updated to "Closed - Converted", automatically create a follow-up Task for the new Contact owner.
3. **Complex Discount Calculation**: When an Opportunity Line Item is added, calculate a complex volume-based discount across multiple related product lines and update the Opportunity total.
4. **Cross-Object Validation**: Before creating a new Case, verify that the related Account's "Support Plan" field is active, otherwise prevent the Case creation.
5. **Sync Data with External System**: After an Order's status changes to "Fulfilled", make an API callout to an external ERP system to update the inventory status.

## 5. Query Examples (English Query Ideas translated to SOQL concepts)
1. **English Idea**: "Find all Accounts in the technology sector that have high revenue."
   - *SOQL Concept*: `SELECT Id, Name FROM Account WHERE Industry = 'Technology' AND AnnualRevenue > 1000000`
2. **English Idea**: "Show me all open Support Cases created this week."
   - *SOQL Concept*: `SELECT Subject, Status FROM Case WHERE Status != 'Closed' AND CreatedDate = THIS_WEEK`
3. **English Idea**: "List all Contacts related to a specific Account named 'Acme Corp'."
   - *SOQL Concept*: `SELECT FirstName, LastName FROM Contact WHERE Account.Name = 'Acme Corp'`
4. **English Idea**: "Find all Opportunities worth more than $50,000 that are expected to close this month."
   - *SOQL Concept*: `SELECT Name, Amount FROM Opportunity WHERE Amount > 50000 AND CloseDate = THIS_MONTH`
5. **English Idea**: "Retrieve all users who have not logged in for the last 30 days."
   - *SOQL Concept*: `SELECT Name, LastLoginDate FROM User WHERE LastLoginDate < LAST_N_DAYS:30`

## 6. Reflection: Why enterprise systems react automatically to data changes
Enterprise systems react automatically to data changes to ensure data consistency, maintain business rules, and improve operational efficiency. Automation removes the reliance on manual human intervention, reducing the risk of errors and delays. For example, automatically updating an inventory level when a sale is closed ensures that the sales team always has accurate stock information, preventing overselling. This real-time reactivity allows organizations to scale operations smoothly, provide faster customer service, and ensure that complex, multi-step business processes are executed consistently every time critical data is modified.
