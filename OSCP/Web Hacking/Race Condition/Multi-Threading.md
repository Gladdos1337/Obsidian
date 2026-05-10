## Programs

A **program** is a set of instructions to achieve a specific task. You need to execute the program to accomplish what you want. Unless you execute it, it won’t do anything and remains a set of static instructions.

Think of it as a recipe; you just downloaded a new coffee recipe that includes a variety of herbs, such as cardamom and cinnamon. These are the instructions:

```markdown
1. Combine brewed coffee, cardamom, cinnamon, and cloves (if using) in a saucepan.
2. Heat the mixture over low heat for 5 minutes, stirring occasionally. Do not boil.
3. Strain the coffee into your mug.
4. Add milk if desired, and sweeten to taste with honey or sugar.
```

```python
# Import the Flask class from the flask module
from flask import Flask

# Create an instance of the Flask class representing the application
app = Flask(__name__)

# Define a route for the root URL ('/')
@app.route('/')
def hello_world():
    # This function will be executed when the root URL is accessed
    # It returns a string containing HTML code for a simple web page
    return '<html><head><title>Greeting</title></head><body><h1>Hello, World!</h1></body></html>'

# This checks if the script is being run directly (as the main program)
# and not being imported as a module
if __name__ == '__main__':
    # Run the Flask application
    # The host='0.0.0.0' allows the server to be accessible from any IP address
    # The port=8080 specifies the port number on which the server will listen
    app.run(host='0.0.0.0', port=8080)
```

## Processes

One afternoon, you decide to try out this new coffee recipe you downloaded online. You start going through the recipe one step at a time. You are in the process of making this coffee recipe. While in the “process” of “executing” the “instructions,” you might get interrupted by an urgent call. Or you might work on another “job” while waiting for the water to heat. Interruptions and waiting are generally unavoidable. The act of carrying out the recipe instructions to make coffee is similar to the process of executing program instructions.

A **process** is a **_program_** in execution. In some literature, you might come across the term **job**. Both terms refer to the same thing, although the term process has superseded the term job. Unlike a program, which is static, a process is a dynamic entity. It holds several key aspects, in particular:

- **Program**: The executable code related to the process
- **Memory**: Temporary data storage
- **State**: A process usually hops between different states. After it is in the New state, i.e., just created, it moves to the Ready state, i.e., ready to run once given CPU time. Once the CPU allocates time for it, it goes to the Running state. Furthermore, it can be in the Waiting state pending I/O or event completion. Once it exits, it moves to the Terminated state.