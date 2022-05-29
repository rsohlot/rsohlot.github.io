# Debug python service in container

Tags: learning, python

What if you have a python service (flask, Django,...) running in a docker container and you want to debug the code.

There is a way to do that `debugpy`

Tools required:

1. vs code
2. docker

Follow the following steps to debug the code:

1. add debugpy to the requirements file

```markdown
debugpy
```

1.  forward the port for debugging from the docker container
    1. If creating a container from Dockerfile then expose the port following way:
    [https://docs.docker.com/engine/reference/builder/#expose](https://docs.docker.com/engine/reference/builder/#expose)
        
        ```docker
        
        EXPOSE 10001
        
        ```
        
    2. If using docker-compose then expose the port following way: 
    
        
        ```yaml
        version: "3.7"
        services:
          python-service:
            container_name: python-service
            build:
              context: .
              args:
                DEV: 1
            command: sh setup/run-backend-docker.sh --reload
            restart: on-failure:5
            volumes:
              - ./:/usr/src/app/
              - /var/run/docker.sock:/var/run/docker.sock    
            ports:
              - 10001:10001
        ```
        
        Here code will decide in `/usr/src/app/` inside docker container.
        
2. Open the code in vs code and create a folder with name `.vscode` in parent folder. And create the following 2 files inside it.
    1. launch.json
        
        ```json
        {
            // Use IntelliSense to learn about possible attributes.
            // Hover to view descriptions of existing attributes.
            // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
            "version": "0.2.0",
            "configurations": [
                { // simple attach to running container - works good
                    "name": "Python Attach",
                    "type": "python",
                    "request": "attach",
                    "port": 10001,
                    "host": "localhost",
                    "pathMappings": [
                        {
                            "localRoot": "${workspaceFolder}/",
                            "remoteRoot": "/usr/src/app/"
                        }
                    ]
                }
            ]
        
        ```
        
    2. tasks.json
    
    ```json
    {
    	"label": "docker-compose-start",
    	"type": "shell",
    	"command": "docker compose -f docker-compose.dev.yml up -d --build -d",
    	"isBackground": true,
    	"problemMatcher": [
    	  {
    		"pattern": [{ "regexp": ".", "file": 1, "location": 2, "message": 3, }],
    		"background": {
    		  "activeOnStart": true,
    		  "beginsPattern": "^(Building py-service)$",
    		  "endsPattern": "^(Creating|Recreating|Starting) (py-container) ... (done)$",
    		}
    	  },
    	],
      }
    ```
    
    Here we are using docker-compose to create a python service. That's why in the command we specified `docker-compose -f docker-compose.dev.yml up -d --build -d`, if you are using Dockerfile use the following command: docker run —name container_name image_name.
    
    Reference: [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)
    
3. Now, let's add a listener which will keep track of your button and enables when you will press F5 (in vs code) to debug and attach the debugger.
For that add the following line before our service app

```python
import multiprocessing

if multiprocessing.current_process().pid > 1:
    import debugpy

    debugpy.listen(("0.0.0.0", 10001))
    print("⏳ VS Code debugger can now be attached, press F5 in VS Code ⏳", flush=True)
    debugpy.wait_for_client()
    print("🎉 VS Code debugger attached, enjoy debugging 🎉", flush=True)

app = create_app()
```

Reference :

1. [https://blog.theodo.com/2020/05/debug-flask-vscode/](https://blog.theodo.com/2020/05/debug-flask-vscode/)
2. [https://medium.com/@lassebenninga/how-to-debug-flask-running-in-docker-compose-in-vs-code-ef37f0f516ee](https://medium.com/@lassebenninga/how-to-debug-flask-running-in-docker-compose-in-vs-code-ef37f0f516ee)