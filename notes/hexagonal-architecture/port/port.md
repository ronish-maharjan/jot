## There are mainly two type of port in **Hexagonal Architecture**
- inbound port 
- outbound port 

## inbound port 
To simply understand what is port first its just a blueprint of some feature that actual feature need to implement ** use interface ** to create the port 
- so now inbound port is the blueprint that can be called by the outsider or the user like suppose we have port called **RegisterUser** it can be called by outside right 

## outbound port 
- so lets continue from where we left off in the **inbound port** so the **RegisterUser** can be called by the user but internally it require some other port blueprint like **saveuser** method which is on the blueprint of UserRepositroy so here the UserRepository is the outbound port 
