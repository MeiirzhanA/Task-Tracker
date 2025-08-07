#!/usr/bin/env python3 https://roadmap.sh/projects/task-tracker
import sys
import json
import os
from datetime import datetime

TASKS_FILE = 'tasks.json'

def load_tasks():
    if not os.path.exists(TASKS_FILE):
        return []
    with open(TASKS_FILE, 'r') as f:
        try:
            return json.load(f)
        except json.JSONDecodeError:
            return []

def save_tasks(tasks):
    with open(TASKS_FILE, 'w') as f:
        json.dump(tasks, f, indent=4)

def generate_id(tasks):
    if not tasks:
        return 1
    return max(task['id'] for task in tasks) + 1

def find_task(tasks, task_id):
    for task in tasks:
        if task['id'] == task_id:
            return task
    return None

def add_task(description):
    tasks = load_tasks()
    task_id = generate_id(tasks)
    now = datetime.now().isoformat()
    new_task = {
        'id': task_id,
        'description': description,
        'status': 'todo',
        'createdAt': now,
        'updatedAt': now
    }
    tasks.append(new_task)
    save_tasks(tasks)
    print(f"Task added successfully (ID: {task_id})")

def update_task(task_id, description):
    tasks = load_tasks()
    task = find_task(tasks, task_id)
    if not task:
        print(f"Task with ID {task_id} not found.")
        return
    task['description'] = description
    task['updatedAt'] = datetime.now().isoformat()
    save_tasks(tasks)
    print(f"Task {task_id} updated successfully.")

def delete_task(task_id):
    tasks = load_tasks()
    task = find_task(tasks, task_id)
    if not task:
        print(f"Task with ID {task_id} not found.")
        return
    tasks = [t for t in tasks if t['id'] != task_id]
    save_tasks(tasks)
    print(f"Task {task_id} deleted successfully.")

def mark_task(task_id, status):
    tasks = load_tasks()
    task = find_task(tasks, task_id)
    if not task:
        print(f"Task with ID {task_id} not found.")
        return
    task['status'] = status
    task['updatedAt'] = datetime.now().isoformat()
    save_tasks(tasks)
    print(f"Task {task_id} marked as {status}.")

def list_tasks(filter_status=None):
    tasks = load_tasks()
    if filter_status:
        filtered = [t for t in tasks if t['status'] == filter_status]
    else:
        filtered = tasks

    if not filtered:
        print("No tasks found.")
        return

    for task in filtered:
        print(f"ID: {task['id']}")
        print(f"Description: {task['description']}")
        print(f"Status: {task['status']}")
        print(f"Created At: {task['createdAt']}")
        print(f"Updated At: {task['updatedAt']}")
        print("-" * 20)

def print_usage():
    usage = """
Usage:
  task-cli add "task description"
  task-cli update <id> "new description"
  task-cli delete <id>
  task-cli mark-in-progress <id>
  task-cli mark-done <id>
  task-cli list [todo|in-progress|done]
"""
    print(usage)

def main():
    if len(sys.argv) < 2:
        print_usage()
        return

    command = sys.argv[1]

    if command == 'add':
        if len(sys.argv) < 3:
            print("Please provide task description.")
            return
        description = sys.argv[2]
        add_task(description)

    elif command == 'update':
        if len(sys.argv) < 4:
            print("Please provide task ID and new description.")
            return
        try:
            task_id = int(sys.argv[2])
        except ValueError:
            print("Task ID must be an integer.")
            return
        description = sys.argv[3]
        update_task(task_id, description)

    elif command == 'delete':
        if len(sys.argv) < 3:
            print("Please provide task ID.")
            return
        try:
            task_id = int(sys.argv[2])
        except ValueError:
            print("Task ID must be an integer.")
            return
        delete_task(task_id)

    elif command == 'mark-in-progress':
        if len(sys.argv) < 3:
            print("Please provide task ID.")
            return
        try:
            task_id = int(sys.argv[2])
        except ValueError:
            print("Task ID must be an integer.")
            return
        mark_task(task_id, 'in-progress')

    elif command == 'mark-done':
        if len(sys.argv) < 3:
            print("Please provide task ID.")
            return
        try:
            task_id = int(sys.argv[2])
        except ValueError:
            print("Task ID must be an integer.")
            return
        mark_task(task_id, 'done')

    elif command == 'list':
        if len(sys.argv) == 3:
            status = sys.argv[2]
            if status not in ['todo', 'in-progress', 'done']:
                print("Invalid status filter. Use: todo, in-progress, or done.")
                return
            list_tasks(status)
        else:
            list_tasks()

    else:
        print(f"Unknown command: {command}")
        print_usage()

if __name__ == "__main__":
    main()
