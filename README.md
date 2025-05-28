# Class Activity - Hexagonal Architechture with Task application

## ✅ Requirements

- Modify the Task entity to support a "done" status.
- Extend the input port (TaskInputPort) with the method `mark_task_done`.
- Update the use case to implement the logic for marking a task as done.
- Expand the repository to allow task lookup and updates.
- Add a new HTTP REST endpoint to expose the functionality:
  - **PUT** `/tasks/<id>/done`  
    Marks the task identified by ID as completed.
---

## Modifications Made

### 1. Modify the Domain

In `entities.py`, there is a method `mark_done()` in the `Task` entity to change the `done` status to `True`.

```python
@dataclass
class Task:
    id: str
    title: str
    done: bool = False

    def mark_done(self):
        self.done = True
```

### 2. Extend the Input Port

In `ports.py`, the abstract method was added:

```python
def mark_task_done(self, task_id: str) -> Task: pass
```

### 3. Update the Use Case

In `use_cases.py`, the logic for marking a task as done was implemented using the repository to find and update the task:

```python
def mark_task_done(self, task_id: str) -> Task:
    task = self.repo.find_by_id(task_id)
    task.mark_done()
    self.repo.update(task)
    return task
```

### 4. Improve the Repository

In `memory_repo.py`, methods were added to find a task by ID and update it:

```python
def find_by_id(self, task_id: str) -> Task:
    for task in self.tasks:
        if task.id == task_id:
            return task
    raise ValueError(f"Task with id {task_id} not found")

def update(self, task: Task) -> None:
    for i, existing_task in enumerate(self.tasks):
        if existing_task.id == task.id:
            self.tasks[i] = task
            return
    raise ValueError(f"Task with id {task.id} not found")
```

### 5. Add New HTTP Endpoint

In `http_handler.py`, a route was added to handle marking tasks as done:

```python
@app.route("/tasks/<task_id>/done", methods=["PUT"])
def mark_task_done(task_id):
    try:
        task = use_case.mark_task_done(task_id)
        return jsonify({"id": task.id, "title": task.title, "done": task.done}), 200
    except ValueError as e:
        return jsonify({"error": str(e)}), 404
```

---

## Usage and Testing

### Create a Task

```bash
curl -X POST http://localhost:5000/tasks \
     -H "Content-Type: application/json" \
     -d '{"title": "Finish challenge"}'
```

Expected response:

```json
{
  "id": "uuid-generated-id",
  "title": "Finish challenge",
  "done": false
}
```

### Mark a Task as Done

Replace `<id>` with the ID returned when creating the task.

```bash
curl -X PUT http://localhost:5000/tasks/<id>/done
```

Expected response:

```json
{
  "id": "<id>",
  "title": "Finish challenge",
  "done": true
}
```

---

## Flow Diagrams

### 1. "Mark Task as Done" Functionality Flow

```mermaid
flowchart TD
    A["Client sends PUT /tasks/{id}/done request"] --> B["HTTP Handler receives the request"]
    B --> C["Executes Use Case: mark_task_done(id)"]
    C --> D["Repository searches task by ID"]
    D -->|Task not found| E["Returns 404 error: 'Task not found'"]
    D -->|Task found| F["Sets done=True in Task entity"]
    F --> G["Repository updates task in database"]
    G --> H["Returns JSON response with updated task"]
    H --> I["Client receives success confirmation"]
```

### 2. Hexagonal Architecture with New Feature

![Hexagonal architecture](Hexagonal arch.jpg)

---

## Conclusion

This enhancement allows updating task status via API while respecting hexagonal architecture, ensuring modularity and scalability.
