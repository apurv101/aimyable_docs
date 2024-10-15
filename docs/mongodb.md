
# MongoDB Database Structure for Aimyable

To design the MongoDB database structure for Aimyable, considering the various components and processes involved, we will break down the requirements into collections and the fields necessary for each document type. The structure is designed to store projects, tasks, instructions, actions, and relevant execution histories. Here's a detailed overview of the MongoDB schema:

### 1. **`projects` Collection**

The `projects` collection stores high-level information about each project created by users. A project can consist of multiple tasks that work towards fulfilling a specific goal.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "project_id": "unique-project-id",
  "user_id": "user-id", 
  "project_description": "Check emails from specific vendors and create bills for each",
  "tasks": [
    {
      "task_id": "task-id-1",
      "status": "pending",
      "updated_at": ISODate("..."),
      "custom_instructions": {
        "instruction": "Use top menu instead of side menu in QuickBooks",
        "created_at": ISODate("...")
      }
    }
  ],
  "schedule": {
    "frequency": "daily",
    "start_time": "2024-10-15T08:00:00Z"
  },
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `project_id`: Unique identifier for the project.
- `user_id`: ID of the user who created the project.
- `project_description`: Description of the project.
- `tasks`: Array of task references related to the project.
- `schedule`: Information about the project’s scheduling (e.g., daily, weekly).
- `created_at`: Timestamp when the project was created.
- `updated_at`: Timestamp when the project was last updated.

### 2. **`tasks` Collection**

The `tasks` collection stores individual tasks that are part of a project. Each task is broken down into instructions.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "task_id": "unique-task-id",
  "project_id": "project-id", 
  "task_description": "Open Outlook and check for emails from xyz@gmail.com",
  "instructions": [
    {
      "instruction_id": "instruction-id-1",
      "status": "pending",
      "updated_at": ISODate("...")
    }
  ],
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `task_id`: Unique identifier for the task.
- `project_id`: Reference to the parent project.
- `task_description`: Description of the task.
- `instructions`: Array of instruction references related to the task.
- `created_at`: Timestamp when the task was created.
- `updated_at`: Timestamp when the task was last updated.

### 3. **`instructions` Collection**

The `instructions` collection contains detailed instructions that specify how a task should be executed. Each instruction is stored in a structured JSON format.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "instruction_id": "unique-instruction-id",
  "task_id": "task-id", 
  "instruction_description": "Open the email from xyz@gmail.com and save the data",
  "engine": "instruction_execution",
  "instruction": {
    "type": "action",
    "details": "save the data from the email"
  },
  "memory_key": "email_by_xyz_2341",
  "custom": "Optional user-specified instructions",
  "action_history": [
    {
      "action": "Click on the inbox",
      "status": "completed",
      "timestamp": ISODate("...")
    }
  ],
  "status": "pending",
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `instruction_id`: Unique identifier for the instruction.
- `task_id`: Reference to the parent task.
- `instruction_description`: High-level description of the instruction.
- `engine`: Specifies which engine will execute this instruction (e.g., `instruction_execution`, `app_opening`).
- `instruction`: Contains the type and details of the instruction.
- `memory_key`: Used to store intermediate data in temporary memory (e.g., MongoDB, Redis).
- `custom`: Optional custom instructions provided by the user.
- `action_history`: An array of actions performed during instruction execution, along with their status.
- `status`: Current status of the instruction (e.g., pending, in-progress, completed, failed).
- `created_at`: Timestamp when the instruction was created.
- `updated_at`: Timestamp when the instruction was last updated.

### 4. **`actions` Collection**

The `actions` collection stores the smallest units of execution. These actions are derived from instructions and performed by the Mini-RPA.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "action_id": "unique-action-id",
  "instruction_id": "instruction-id",
  "description": "Click on the Outlook inbox",
  "status": "completed",
  "result": {
    "screenshot": "screenshot-path",
    "feedback": "positive"
  },
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `action_id`: Unique identifier for the action.
- `instruction_id`: Reference to the parent instruction.
- `description`: High-level description of the action.
- `status`: Current status of the action (e.g., pending, in-progress, completed, failed).
- `result`: Contains the screenshot path or any relevant feedback after action execution.
- `created_at`: Timestamp when the action was created.
- `updated_at`: Timestamp when the action was last updated.

### 5. **`users` Collection**

The `users` collection stores user-related information, including projects they have created and their preferences for task execution.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "user_id": "unique-user-id",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "projects": [
    {
      "project_id": "project-id-1",
      "created_at": ISODate("...")
    }
  ],
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `user_id`: Unique identifier for the user.
- `name`: Full name of the user.
- `email`: User's email address.
- `projects`: Array of project references related to the user.
- `created_at`: Timestamp when the user was created.
- `updated_at`: Timestamp when the user information was last updated.

### 6. **`conversational_history` Collection**

This collection stores interactions between users and the system, capturing the conversational history for future retrieval and use in context-aware actions.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "user_id": "user-id",
  "conversation_id": "unique-conversation-id",
  "interaction": "What project needs to be done today?",
  "response": "Check emails from xyz@gmail.com and create bills.",
  "created_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `user_id`: Reference to the user who initiated the interaction.
- `conversation_id`: Unique identifier for the conversation.
- `interaction`: The user's prompt or input.
- `response`: The system's response to the interaction.
- `created_at`: Timestamp when the interaction took place.

### 7. **`temporary_storage` Collection**

The `temporary_storage` collection handles the short-term storage of intermediate results, such as email content or extracted data, used across different instructions.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "memory_key": "email_by_xyz_2341",
  "data": {
    "email_content": "Invoice from xyz@gmail.com",
    "attachments": ["invoice.pdf"]
  },
  "created_at": ISODate("..."),
  "expires_at": ISODate("...")
}
```

**Fields:**
- `_id`: MongoDB’s unique identifier.
- `memory_key`: Key used to reference the stored data in subsequent instructions.
- `data`: Stores the temporary data (e.g., email content, attachments).
- `created_at`: Timestamp when the data was stored.
- `expires_at`: Timestamp when the data will expire and be purged.

### 8. **`task_scheduling` Collection**

The `task_scheduling` collection stores scheduling information for projects and tasks, allowing the system to trigger them at predefined times.

**Document Structure:**

```json
{
  "_id": ObjectId("..."),
  "schedule_id": "unique-schedule-id",
  "project_id": "project-id",
  "user_id": "user-id",
  "schedule": {
    "frequency": "daily",
    "time": "08:00",
    "next_execution": ISODate("2024-10-16T08:00:00Z")
  },
  "created_at": ISODate("..."),
  "updated_at": ISODate("...")
}
```

**Fields:**
- `_
