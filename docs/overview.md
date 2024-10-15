# Overview of Aimyable Architecture

## 1. User Interface Layer
The User Interface (UI) Layer is the primary interaction point for users. There are two core functionalities:

### 1.1 Predefined Projects & Scheduling
- Users can create **predefined projects**, which are large collections of tasks.
- These projects can be **scheduled** to run at specific times, such as weekly or daily.
- **Projects** are broken down into tasks, which are then decomposed into executable instructions by various engines.
- Example Project: "Check all emails from specific vendors and create bills." This project is split into individual tasks like "Check email by a specific vendor and create a bill."

### 1.2 Dynamic Queries & Fine-Tuned Instructions
- Users can ask for information about ongoing or upcoming projects, such as “What project needs to be done today?”
- Users can **modify schedules** dynamically.
- Users can provide **general tasks**, such as “Read an email and create a bill,” or fine-tune **specific task instructions**, like: "Instead of using the side menu in QuickBooks, use the top menu."

## 2. Task to Instruction Conversion
Once a project is broken down into tasks, each task is converted into a set of JSON-based instructions. The instructions follow this format:

```json
{
    "engine": "<engine_name>",
    "instruction": "<instruction_details>",
    "memory_key": "<key_for_temporary_storage>",
    "custom": "<optional_custom_instruction>"
}
```

### Example:
For the task "Read an email from xyz@gmail.com dated 10/21/2024 and create a bill in QuickBooks", the instructions are:

1. **Opening Outlook:**
   ```json
   {"engine":"app_opening", "instruction":"outlook"}
   ```

2. **Accessing Email:**
   ```json
   {"engine":"instruction_execution", "instruction":"open the email from xyz@gmail.com and save the data", "memory_key": "email_by_xyz_2341"}
   ```
   - The email content is stored temporarily in MongoDB.

3. **Opening QuickBooks:**
   ```json
   {"engine":"app_opening", "instruction":"QuickBooks", "type":"desktop_app"}
   ```

4. **Creating a Bill in QuickBooks:**
   ```json
   {"engine":"instruction_execution", "instruction":"create bill for xyz@gmail.com", "memory_key": "email_by_xyz_2341"}
   ```

## 3. Instruction Execution Engine (Previously Task Execution Engine)
The **Instruction Execution Engine** is responsible for executing instructions derived from tasks. Components include:

- **YOLO & Google Vision**: Used to detect UI elements such as icons and buttons on the screen.
- **LLMs (Large Language Models)**: Handle natural language processing tasks and generate text-based actions.
- **Memory Handling**: Data is stored temporarily in Redis and passed between steps.
- **Custom Instructions**: Users can specify detailed instructions, such as "Take a screenshot" or "Use top menu instead of side menu," which will be processed by the relevant engines.

## 4. App Opening Engine
The **App Opening Engine** handles opening desktop or web apps as part of task execution.

- **YOLO Detection**: A dedicated YOLO model detects app icons on the screen.
- **Desktop vs. Web App Handling**: Instructions differentiate between desktop and web apps:
  - `"instruction": "QuickBooks", "type": "desktop_app"`
  - `"instruction": "Gmail", "type": "web_app"`

## 5. Mini-RPA
The **Mini-RPA** is deployed on a desktop and interacts with the system through Celery queues.

- **Queue**: Tasks are broken into instructions and queued for processing.
- **Screenshot Handling**: Mini-RPA captures screenshots, sends them to the server for action prediction, and then executes the predicted actions.
- **Feedback Loop**: After executing actions, the system takes new screenshots to ensure it reaches the correct state.

## 6. Server & Processing Components
The server hosts all the engines and manages processing:

- **Instruction Execution Engine & App Opening Engine**: These engines process instructions on the server.
- **Vision Models & Prompt Engines**: Vision models (YOLO, Google Vision) and prompts predict the next action based on screenshots.
- **Temporary Storage**: MongoDB is used to store intermediate data such as extracted emails during task execution.
- **Caching**: Frequently repeated tasks are cached for faster processing.

## 7. Task Decomposition & Customization
The **Task Decomposition Engine** breaks down tasks into instructions while considering user preferences.

- **User-Specified Instructions**: Users can provide alternative paths or ways to perform tasks, like “Use top menu instead of side menu.”
- **Custom Instructions**: Custom instructions are stored under the `"custom"` key in the JSON format and passed to the relevant engines.

## 8. Database & Vector Storage
- **Conversation History Storage**: All user interactions are stored in a vector database, allowing for easy retrieval and context continuity.
- **Task & Instruction Storage**: Tasks, instructions, and interactions are stored for reuse and reporting.

## 9. DevOps & Infrastructure
The architecture is designed for scalability and continuous operation:

- **MongoDB**: Task instructions are distributed across engines for asynchronous processing.
- **Server-Client Communication**: Mini-RPA communicates with the server for real-time execution of actions.
- **Vector Database**: Provides fast retrieval of past user conversations and task details.
  
## 10. Scheduling & Querying
- **Project Scheduling**: Projects can be scheduled to run at specific times.
- **User Queries**: Users can inquire about the status of projects, change schedules, or request new tasks or modifications.

## Key Features
- **Fine-Tuned Instruction Handling**: Users can modify task execution paths with custom commands, passed through all engine layers.
- **Low-Level Commands**: Actions such as "Take a screenshot" are executed at the instruction level, giving users precise control.
- **Memory & Caching**: MongoDB is used to store intermediate results, and a caching system helps optimize repeated task execution.

This architecture provides a seamless, flexible solution for managing projects, breaking them down into tasks, and converting those tasks into executable instructions while supporting fine-tuned user control.
