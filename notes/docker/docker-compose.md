# Notes for what i learned while creating the docker compose file 

## Things to remember
> docker will generate name for container based on projectname_servicename_index
> created volume name should be match the used volume name
> ${} used to access the variables from the .env file of the projects
> port map should be inside the quotation ":" and in this format "system_port:container_port"
> use space to key value in yml  eg name: "ronish" not name:"ronish"

## Format or syntax you can say for writing the yml 
```bash 
    name: postgres_testing (this is the project name and is added as prefix for the volume name and if the container name is not added it will be added by default)

    services: it contain all the configuration required for the containers basically 

        postgres: this is the service name

            image: postgres:16-alpine (specify what image to run on the container)

            container_name: postgres_db (what should be the contaienr name)

            restart: unless-stopped (specify when should it restart it have basically 3 types `always`, `unless-stopped`, `on-failure` )

            environment: (used to add environment variables)
                varaiblename: ${env_file_variable_name}
                varaiblename: ${env_file_variable_name}

            ports:  (map the container port with the system port)
                - "5432:5432"

            volumes: here we attached the created volume
                - postgresData:/var/lib/postgres/data 

volumes: here what we create the volume 
    - postgresData: 


```
