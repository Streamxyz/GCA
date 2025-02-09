# GCA
Cloud Agents
Google Cloud Services for storage and data management while keeping the Qwen models and using Visual Studio Code (VS Code) for development, you'll need to make some adjustments to the architecture and code. Below is a comprehensive guide that outlines the necessary changes.
Project Architecture
Task Manager Agent:
Responsible for managing and assigning tasks to other agents.
Prompt Engineer Agent:
Translates user tasks into well-written prompts for the LLM.
Coder Agent:
Executes custom code using the Python Code Interpreter tool.
QA Agent:
Provides quality assurance on the outputs.
Knowledge Base:
Stores and retrieves domain-specific knowledge to enhance the agents' capabilities.
Uses Google Cloud Storage (GCS) for storing documents.
Uses Google Cloud Firestore for structured data and metadata.
Multi-Agent Orchestration Application:
Coordinates the Task Manager, Prompt Engineer, Coder, and QA agents.
Workflow Application:
Automates the process of task assignment, code execution, prompt translation, and QA.
File and Data Management:
Uses Google Cloud Storage (GCS) to manage files and data.
Uses Google Cloud Firestore for structured data and metadata.
File Structure
Copy
1
2
3
4
5
6
7
8
9
10
/project-root
   /src
       main.py          # Main application file
       agents.py        # Definitions of Task Manager, Prompt Engineer, Coder, and QA agents
       tools.py         # Definitions of tools (e.g., Python Code Interpreter)
       utils.py         # Utility functions for file and data management using Google Cloud Services
       config.py        # Configuration settings for API keys and other constants
   /data
       knowledge_base   # Directory for knowledge base files
   /requirements.txt    # Dependency list for Python packages
Code Implementation
config.py
Contains configuration settings such as API keys, model details, and Google Cloud credentials.
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
# config.py
import os


# API Keys
DASHSCOPE_API_KEY = os.getenv('DASHSCOPE_API_KEY')
GOOGLE_APPLICATION_CREDENTIALS = os.getenv('GOOGLE_APPLICATION_CREDENTIALS')


# Model Details
MODEL_NAME = 'qwen-plus'


# Google Cloud Storage Bucket Name
GCS_BUCKET_NAME = 'your-gcs-bucket-name'  # Replace with your actual GCS bucket name


# Google Cloud Firestore Collection Name
FIRESTORE_COLLECTION_NAME = 'your-firestore-collection-name'  # Replace with your actual Firestore collection name
tools.py
Defines tools that the agents can use, such as the Python Code Interpreter.
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
⌄
⌄
⌄
# tools.py
import json
from dashscope import Application


def code_interpreter(code):
   response = Application.call(
       app_id='YOUR_CODE_INTERPRETER_APP_ID',  # Replace with your actual app ID
       prompt=f'Execute the following Python code:\n{code}',
       api_key=os.getenv('DASHSCOPE_API_KEY')
   )
   if response.status_code != 200:
       return f"Error: {response.message}"
   else:
       return response.output['text']
agents.py
Creates and configures the Task Manager, Prompt Engineer, Coder, and QA agents.
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
# agents.py
from dashscope import Assistants
from config import MODEL_NAME


def create_task_manager():
   task_manager = Assistants.create(
       model=MODEL_NAME,
       name='Task Manager',
       description='An agent responsible for managing and assigning tasks.',
       instructions='You are a task manager. Assign tasks to the Coder, Prompt Engineer, and QA agents based on the user input.',
       tools=[]
   )
   return task_manager


def create_prompt_engineer():
   prompt_engineer = Assistants.create(
       model=MODEL_NAME,
       name='Prompt Engineer',
       description='An agent responsible for translating tasks into well-written prompts.',
       instructions='You are a prompt engineer. Translate the provided task into a well-written prompt for the LLM.',
       tools=[]
   )
   return prompt_engineer


def create_coder():
   coder = Assistants.create(
       model=MODEL_NAME,
       name='Coder',
       description='An agent responsible for executing custom code.',
       instructions='You are a coder. Execute the provided code and return the results.',
       tools=[
           {
               'type': 'function',
               'function': {
                   'name': 'code_interpreter',
                   'description': 'Executes Python code snippets.',
                   'parameters': {
                       'type': 'object',
                       'properties': {
                           'code': {
                               'type': 'string',
                               'description': 'The Python code to be executed.'
                           }
                       },
                       'required': ['code']
                   }
               }
           }
       ]
   )
   return coder


def create_qa():
   qa = Assistants.create(
       model=MODEL_NAME,
       name='QA',
       description='An agent responsible for quality assurance.',
       instructions='You are a QA agent. Review the provided output and ensure it meets the required standards.',
       tools=[]
   )
   return qa
utils.py
Handles file and data management using Google Cloud Storage (GCS) and Firestore.
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
⌄
⌄
⌄
⌄
⌄
⌄
# utils.py
from google.cloud import storage, firestore
import os


def initialize_gcs_client():
   gcs_client = storage.Client.from_service_account_json(os.getenv('GOOGLE_APPLICATION_CREDENTIALS'))
   return gcs_client


def initialize_firestore_client():
   firestore_client = firestore.Client.from_service_account_json(os.getenv('GOOGLE_APPLICATION_CREDENTIALS'))
   return firestore_client


def upload_file_to_gcs(gcs_client, bucket_name, file_path, local_file_path):
   bucket = gcs_client.bucket(bucket_name)
   blob = bucket.blob(file_path)
   blob.upload_from_filename(local_file_path)


def download_file_from_gcs(gcs_client, bucket_name, file_path, local_file_path):
   bucket = gcs_client.bucket(bucket_name)
   blob = bucket.blob(file_path)
   blob.download_to_filename(local_file_path)


def add_document_to_firestore(firestore_client, collection_name, document_data):
   collection_ref = firestore_client.collection(collection_name)
   collection_ref.add(document_data)


def get_documents_from_firestore(firestore_client, collection_name):
   collection_ref = firestore_client.collection(collection_name)
   docs = collection_ref.stream()
   return [doc.to_dict() for doc in docs]
main.py
Main application file that initializes agents, sets up the workflow, and handles user interactions.
python
Copy
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
⌄
# main.py
       text_conversion_node,
       end_node
   ],
   edges=[
       {'source': start_node.id, 'target': task_manager_node.id},
       {'source': task_manager_node.id, 'target': prompt_engineer_node.id},
       {'source': prompt_engineer_node.id, 'target': coder_node.id},
       {'source': coder_node.id, 'target': qa_node.id},
       {'source': qa_node.id, 'target': text_conversion_node.id},
       {'source': text_conversion_node.id, 'target': end_node.id}
   ]
)


def handle_function_calling(run):
   if run.required_action:
       for tool_call in run.required_action.submit_tool_outputs.tool_calls:
           if tool_call.function.name == "code_interpreter":
               args = json.loads(tool_call.function.arguments)
               result = code_interpreter(args["code"])
               Runs.submit_tool_outputs(
                   thread_id=run.thread_id,
                   run_id=run.id,
                   tool_outputs=[{"tool_call_id": tool_call.id, "output": result}]
               )
               run = Runs.wait(thread_id=run.thread_id, run_id=run.id)
   return run


def test_workflow(prompt):
   # Create a new thread
   thread = Threads.create()
   # Add user message to the thread
   Messages.create(
       thread_id=thread.id,
       role="user",
       content=prompt
   )
   # Run the workflow
   run = Runs.create(thread_id=thread.id, workflow_id=workflow.id)
   # Wait for the run to complete
   run = Runs.wait(thread_id=thread.id, run_id=run.id)
   # Handle function calling if needed
   run = handle_function_calling(run)
   # Get the assistant's response
   messages = Messages.list(thread_id=thread.id)
   for message in messages.data:
       if message.role == "assistant":
           print(f"Assistant: {message.content[0].text.value}")


if __name__ == '__main__':
   # Example prompt
   prompt = "Calculate the Fibonacci sequence up to 10 using Python."
   test_workflow(prompt)
requirements.txt
Lists the required Python packages.
plaintext
Copy
1
2
3
dashscope
google-cloud-storage
google-cloud-firestore
Explanation of the Code
config.py:
Contains configuration settings for API keys, model details, and Google Cloud credentials.
tools.py:
Defines the code_interpreter tool that executes Python code snippets using a dedicated workflow application.
agents.py:
Creates and configures the Task Manager, Prompt Engineer, Coder, and QA agents using the DashScope SDK.
Each agent is initialized with a specific model and instructions.
The Coder agent includes the code_interpreter tool.
utils.py:
Handles file and data management using Google Cloud Storage (GCS) and Firestore.
Initializes the GCS and Firestore clients and provides functions to upload and download files, as well as add and retrieve documents from Firestore.
main.py:
Initializes the GCS and Firestore clients and agents.
Creates an agent group and workflow nodes.
Connects the workflow nodes to form a complete workflow.
Defines the handle_function_calling function to handle tool calls, specifically the code_interpreter tool.
Defines the test_workflow function to test the workflow with a given prompt.
Runs the workflow with an example prompt to demonstrate the functionality.
Steps to Deploy the Project
Set Up Environment:
Install the required Python packages:
bash
Copy
1
pip install -r requirements.txt
Set environment variables for API keys and Google Cloud credentials:
bash
Copy
1
2
export DASHSCOPE_API_KEY=YOUR_DASHSCOPE_API_KEY
export GOOGLE_APPLICATION_CREDENTIALS=path/to/your/service-account-file.json
Create Workflow Applications:
In Model Studio, create a workflow application for the code_interpreter tool.
Note the application ID and replace YOUR_CODE_INTERPRETER_APP_ID in tools.py.
Create Google Cloud Storage Bucket and Firestore Collection:
In Google Cloud Console, create a GCS bucket to store files and data.
Note the bucket name and replace your-gcs-bucket-name in utils.py.
Create a Firestore collection to store structured data and metadata.
Note the collection name and replace your-firestore-collection-name in utils.py.
Run the Application:
Execute the main.py script to test the workflow and agents.
bash
Copy
1
python src/main.py
Links to Official Documentation
DashScope SDK
Model Studio
Google Cloud Storage
Google Cloud Firestore
Additional Setup for Google Cloud Services
Google Cloud Storage:
Create a GCS bucket.
Upload files to the bucket using the upload_file_to_gcs function in utils.py.
Google Cloud Firestore:
Create a Firestore collection.
Add structured data and metadata to the collection using the add_document_to_firestore function in utils.py.
Retrieve documents from the collection using the get_documents_from_firestore function in utils.py.
Example Code for Google Cloud Services
Uploading a File to GCS
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
⌄
# Example of uploading a file to GCS in Python
from google.cloud import storage
import os


def upload_file_to_gcs(gcs_client, bucket_name, file_path, local_file_path):
   bucket = gcs_client.bucket(bucket_name)
   blob = bucket.blob(file_path)
   blob.upload_from_filename(local_file_path)


# Initialize GCS client
gcs_client = storage.Client.from_service_account_json(os.getenv('GOOGLE_APPLICATION_CREDENTIALS'))


# Upload a file
upload_file_to_gcs(gcs_client, GCS_BUCKET_NAME, 'path/in/bucket/file.txt', 'local/path/to/file.txt')
Adding a Document to Firestore
python
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
⌄
⌄
# Example of adding a document to Firestore in Python
from google.cloud import firestore
import os


def add_document_to_firestore(firestore_client, collection_name, document_data):
   collection_ref = firestore_client.collection(collection_name)
   collection_ref.add(document_data)


# Initialize Firestore client
firestore_client = firestore.Client.from_service_account_json(os.getenv('GOOGLE_APPLICATION_CREDENTIALS'))


# Add a document
document_data = {
   'field1': 'value1',
   'field2': 'value2'
}
add_document_to_firestore(firestore_client, FIRESTORE_COLLECTION_NAME, document_data)
Conclusion
By following these steps, you can design and deploy a team of AI-powered agents using Alibaba Cloud Model Studio and DashScope SDK, while leveraging Google Cloud Storage (GCS) and Firestore for file and data management. This setup ensures that the agents can execute code, manage files and data, and provide code solutions using Alibaba Cloud services.

