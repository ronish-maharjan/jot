## Transaction 

- Transaction is a feature provided by db that helps to group query and run then as one single block we can say that.

**Why it is usefull** 
- Because of how it run the query. It have following feature 
    - If one fail all Fail and only success if all query inside the transaction is succeed.
    - this make the data in the db more accurate

- But still there are some rules we need to follow in transaction which is **ACID (Atomicity,Consistency,Isolation, Durability)**
- By default what i talked all fail if one fail is **Atomicity** so we can say that **Transaction automatically follow Acid property A part**
- But for other rule we have different method , ways to implement in the transaction.
