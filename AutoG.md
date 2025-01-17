# main.py

```python
# main.py

import streamlit as st

from base_models.project_base_model import ProjectBaseModel
from session_variables import initialize_session_variables
from utils.display_util import sidebar_begin, display_main


def main():
    st.set_page_config(page_title="AutoGrok™")
    initialize_session_variables()
    sidebar_begin()
    
    # Main content area
    display_main()


if __name__ == "__main__":
    main()
```

# session_variables.py

```python
# session_variables.py

import streamlit as st

from base_models.project_base_model import ProjectBaseModel
from configs.config_local import LLM_PROVIDER

def initialize_session_variables():

    if "available_models" not in st.session_state:
        st.session_state.available_models = []

    if "current_agent" not in st.session_state:
        st.session_state.current_agent = None

    if "current_project" not in st.session_state:
        st.session_state.current_project = None

    if "current_tool" not in st.session_state:
        st.session_state.current_tool = None

    if "current_workflow" not in st.session_state:
        st.session_state.current_workflow = None

    if "default_llm" not in st.session_state:
        st.session_state.default_llm = ""

    if "default_provider" not in st.session_state:
        st.session_state.default_provider = LLM_PROVIDER

    if "default_provider_key" not in st.session_state:
        st.session_state.default_provider_key = None

    if "project_dropdown" not in st.session_state:
        st.session_state.project_dropdown = "Select..."

    if "project_model" not in st.session_state:
        st.session_state.project_model = ProjectBaseModel()
        
    if "project_name_input" not in st.session_state:
        st.session_state.project_name_input = ""

    if "provider" not in st.session_state:
        st.session_state.provider = ""

    if "reengineer" not in st.session_state:
        st.session_state.reengineer = True

    if "tool_name_input" not in st.session_state:
        st.session_state.tool_name_input = ""        

    if "workflow_dropdown" not in st.session_state:
        st.session_state.workflow_dropdown = "Select..."

    if "workflow_name_input" not in st.session_state:
        st.session_state.workflow_name_input = ""
```

# base_models\agent_base_model.py

```python
# agent_base_model

import os
import yaml

from base_models.tool_base_model import ToolBaseModel
from typing import List, Dict, Optional, Callable


class AgentBaseModel:
    def __init__(
        self,
        name: str,
        config: Dict,
        skills: List[Dict],
        role: str = "",
        goal: str = "",
        backstory: str = "",
        id: Optional[int] = None,
        created_at: Optional[str] = None,
        updated_at: Optional[str] = None,
        user_id: Optional[str] = None,
        workflows: Optional[str] = None,
        type: Optional[str] = None,
        models: Optional[List[Dict]] = None,
        verbose: Optional[bool] = False,
        allow_delegation: Optional[bool] = True,
        new_description: Optional[str] = None,
        timestamp: Optional[str] = None,
        is_termination_msg: Optional[bool] = None,
        code_execution_config: Optional[Dict] = None,
        llm: Optional[str] = None,
        function_calling_llm: Optional[str] = None,
        max_iter: Optional[int] = 25,
        max_rpm: Optional[int] = None,
        max_execution_time: Optional[int] = None,
        step_callback: Optional[Callable] = None,
        cache: Optional[bool] = True
    ):
        self.id = id
        self.name = name
        self.config = config
        self.description = config.get("description", "")
        self.skills = [ToolBaseModel(
            name=skill["title"],
            title=skill["title"],
            content=skill["content"],
            file_name=skill["file_name"],
            description=skill.get("description"),
            timestamp=skill.get("timestamp"),
            user_id=skill.get("user_id")
        ) for skill in skills]
        self.role = role
        self.goal = goal
        self.backstory = backstory
        self.created_at = created_at
        self.updated_at = updated_at
        self.user_id = user_id
        self.workflows = workflows
        self.type = type
        self.models = models
        self.verbose = verbose
        self.allow_delegation = allow_delegation
        self.new_description = new_description
        self.timestamp = timestamp
        self.is_termination_msg = is_termination_msg
        self.code_execution_config = code_execution_config
        self.llm = llm
        self.function_calling_llm = function_calling_llm
        self.max_iter = max_iter
        self.max_rpm = max_rpm
        self.max_execution_time = max_execution_time
        self.step_callback = step_callback
        self.cache = cache


    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "config": {
                "name": self.name,
                "description": self.description,
                # ... other config values ...
            },
            "skills": [skill.to_dict() for skill in self.skills],
            "role": self.role,
            "goal": self.goal,
            "backstory": self.backstory,
            "created_at": self.created_at,
            "updated_at": self.updated_at,
            "user_id": self.user_id,
            "workflows": self.workflows,
            "type": self.type,
            "models": self.models,
            "verbose": self.verbose,
            "allow_delegation": self.allow_delegation,
            "new_description": self.new_description,
            "timestamp": self.timestamp,
            "is_termination_msg": self.is_termination_msg,
            "code_execution_config": self.code_execution_config,
            "llm": self.llm,
            "function_calling_llm": self.function_calling_llm,
            "max_iter": self.max_iter,
            "max_rpm": self.max_rpm,
            "max_execution_time": self.max_execution_time,
            "step_callback": self.step_callback,
            "cache": self.cache
        }


    @classmethod
    def from_dict(cls, data: Dict):
        return cls(
            id=data.get("id"),
            name=data["config"]["name"],
            config=data["config"],
            skills=data["skills"],
            role=data.get("role", ""),
            goal=data.get("goal", ""),
            backstory=data.get("backstory", ""),
            created_at=data.get("created_at"),
            updated_at=data.get("updated_at"),
            user_id=data.get("user_id"),
            workflows=data.get("workflows"),
            type=data.get("type"),
            models=data.get("models"),
            verbose=data.get("verbose", False),
            allow_delegation=data.get("allow_delegation", True),
            new_description=data.get("new_description"),
            timestamp=data.get("timestamp"),
            is_termination_msg=data.get("is_termination_msg"),
            code_execution_config=data.get("code_execution_config"),
            llm=data.get("llm"),
            function_calling_llm=data.get("function_calling_llm"),
            max_iter=data.get("max_iter", 25),
            max_rpm=data.get("max_rpm"),
            max_execution_time=data.get("max_execution_time"),
            step_callback=data.get("step_callback"),
            cache=data.get("cache", True)
        )
    
    @classmethod
    def create_agent(cls, agent_name: str, agent_data: Dict) -> "AgentBaseModel":
        agent = cls.from_dict(agent_data)
        
        # Create a YAML file for the agent
        with open(f"agents/{agent_name}.yaml", "w") as file:
            yaml.dump(agent_data, file)
        
        return agent

    @classmethod
    def get_agent(cls, agent_name: str) -> "AgentBaseModel":
        file_path = f"agents/{agent_name}.yaml"
        if os.path.exists(file_path):
            with open(file_path, "r") as file:
                agent_data = yaml.safe_load(file)
                return cls.from_dict(agent_data)
        else:
            raise FileNotFoundError(f"Agent file not found: {file_path}")

    @staticmethod
    def load_agents() -> List[str]:
        agent_names = []
        for file in os.listdir("agents"):
            if file.endswith(".yaml"):
                agent_name = file[:-5]  # Remove the ".yaml" extension
                agent_names.append(agent_name)
        return agent_names
    
```

# base_models\project_base_model.py

```python
# project_base_model.py

import enum
import os
import yaml

from datetime import datetime
from typing import List, Dict, Optional

class ProjectBaseModel:
    def __init__(
        self,
        prompt: str = "",
        deliverables: List[Dict] = None,
        id: Optional[int] = None,
        created_at: Optional[str] = None,
        updated_at: Optional[str] = None,
        user_id: Optional[str] = None,
        name: Optional[str] = None,
        description: Optional[str] = None,
        status: Optional[str] = None,
        due_date: Optional[str] = None,
        priority: Optional[str] = None,
        tags: Optional[List[str]] = None,
        attachments: Optional[List[str]] = None,
        notes: Optional[str] = None,
        collaborators: Optional[List[str]] = None,
        tools: Optional[List[Dict]] = None,
        workflows: Optional[List[Dict]] = None
    ):
        self.id = id or 1
        self.prompt = prompt
        self.deliverables = deliverables or []
        self.created_at = created_at or datetime.now().isoformat()
        self.updated_at = updated_at
        self.user_id = user_id or "user"
        self.name = name or "project"
        self.description = description
        self.status = status or "not started"
        self.due_date = due_date
        self.priority = priority or "none"
        self.tags = tags or []
        self.attachments = attachments or []
        self.notes = notes
        self.collaborators = collaborators or []
        self.tools = tools or []
        self.workflows = workflows or []

    def add_deliverable(self, deliverable: str):
        self.deliverables.append({"text": deliverable, "done": False})

    @classmethod
    def create_project(cls, project_name: str) -> "ProjectBaseModel":
        project = cls(name=project_name)
        
        # Create a YAML file for the project
        project_data = project.to_dict()
        with open(f"projects/{project_name}.yaml", "w") as file:
            yaml.dump(project_data, file)
        
        return project
    

    @classmethod
    def get_project(cls, project_name: str) -> "ProjectBaseModel":
        file_path = f"projects/{project_name}.yaml"
        if os.path.exists(file_path):
            with open(file_path, "r") as file:
                project_data = yaml.safe_load(file)
                return cls.from_dict(project_data)
        else:
            raise FileNotFoundError(f"Project file not found: {file_path}")
            

    @staticmethod
    def load_projects() -> List[str]:
        project_names = []
        for file in os.listdir("projects"):
            if file.endswith(".yaml"):
                project_name = file[:-5]  # Remove the ".yaml" extension
                project_names.append(project_name)
        return project_names

    def mark_deliverable_done(self, index: int):
        if 0 <= index < len(self.deliverables):
            self.deliverables[index]["done"] = True

    def mark_deliverable_undone(self, index: int):
        if 0 <= index < len(self.deliverables):
            self.deliverables[index]["done"] = False

    def set_prompt(self, prompt: str):
        self.prompt = prompt

    def to_dict(self):
        return {
            "id": self.id,
            "prompt": self.prompt,
            "deliverables": self.deliverables,
            "created_at": self.created_at,
            "updated_at": self.updated_at,
            "user_id": self.user_id,
            "name": self.name,
            "description": self.description,
            "status": self.status,
            "due_date": self.due_date,
            "priority": self.priority,
            "tags": self.tags,
            "attachments": self.attachments,
            "notes": self.notes,
            "collaborators": self.collaborators,
            "tools": self.tools,
            "workflows": self.workflows
        }

    @classmethod
    def from_dict(cls, data: Dict):
        return cls(
            id=data.get("id"),
            prompt=data.get("prompt", ""),
            deliverables=data.get("deliverables", []),
            created_at=data.get("created_at"),
            updated_at=data.get("updated_at"),
            user_id=data.get("user_id"),
            name=data.get("name"),
            description=data.get("description"),
            status=data.get("status"),
            due_date=data.get("due_date"),
            priority=data.get("priority"),
            tags=data.get("tags", []),
            attachments=data.get("attachments", []),
            notes=data.get("notes"),
            collaborators=data.get("collaborators", []),
            tools=data.get("tools", []),
            workflows=data.get("workflows", [])
        )
    

class ProjectPriority(enum.Enum):
    NONE = "none"
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    URGENT = "urgent"


class ProjectStatus(enum.Enum):
    NOT_STARTED = "not started"
    ON_HOLD = "on hold"
    IN_PROGRESS = "in progress"
    COMPLETE = "complete"
    CANCELED = "canceled"
```

# base_models\tool_base_model.py

```python
# tool_base_model.py
import os
import yaml

from typing import List, Dict, Optional

class ToolBaseModel:
    def __init__(
        self,
        name: str,
        title: str,
        content: str,
        file_name: str,
        description: Optional[str] = None,
        timestamp: Optional[str] = None,
        user_id: Optional[str] = None
    ):
        self.name = name
        self.title = title
        self.content = content
        self.file_name = file_name
        self.description = description
        self.timestamp = timestamp
        self.user_id = user_id

    def to_dict(self):
        return {
            "name": self.name,
            "title": self.title,
            "content": self.content,
            "file_name": self.file_name,
            "description": self.description,
            "timestamp": self.timestamp,
            "user_id": self.user_id
        }

    @classmethod
    def from_dict(cls, data: Dict):
        return cls(
            name=data["name"],
            title=data["title"],
            content=data["content"],
            file_name=data["file_name"],
            description=data.get("description"),
            timestamp=data.get("timestamp"),
            user_id=data.get("user_id")
        )
    
    @classmethod
    def create_tool(cls, tool_name: str, tool_data: Dict) -> "ToolBaseModel":
        tool = cls.from_dict(tool_data)
        
        # Create a YAML file for the tool
        with open(f"tools/{tool_name}.yaml", "w") as file:
            yaml.dump(tool_data, file)
        
        return tool

    @classmethod
    def get_tool(cls, tool_name: str) -> "ToolBaseModel":
        file_path = f"tools/{tool_name}.yaml"
        if os.path.exists(file_path):
            with open(file_path, "r") as file:
                tool_data = yaml.safe_load(file)
                return cls.from_dict(tool_data)
        else:
            raise FileNotFoundError(f"Tool file not found: {file_path}")
            
    @staticmethod
    def load_tools() -> List[str]:
        tool_names = []
        for file in os.listdir("tools"):
            if file.endswith(".yaml"):
                tool_name = file[:-5]  # Remove the ".yaml" extension
                tool_names.append(tool_name)
        return tool_names
    
```

# base_models\workflow_base_model.py

```python
# workflow_base_model

import os
import yaml

from base_models.agent_base_model import AgentBaseModel
from configs.config import DEBUG
from datetime import datetime
from typing import List, Dict, Optional


class Sender:
    def __init__(
        self,
        type: str,
        config: Dict,
        timestamp: str,
        user_id: str,
        tools: List[Dict],
    ):
        self.type = type
        self.config = config
        self.timestamp = timestamp
        self.user_id = user_id
        self.tools = tools

    def to_dict(self):
        return {
            "type": self.type,
            "config": self.config,
            "timestamp": self.timestamp,
            "user_id": self.user_id,
            "tools": self.tools,
        }

    @classmethod
    def from_dict(cls, data: Dict):
        return cls(
            type=data["type"],
            config=data["config"],
            timestamp=data["timestamp"],
            user_id=data["user_id"],
            tools=data["tools"],
        )
    

class Receiver:
    def __init__(
        self,
        type: str,
        config: Dict,
        groupchat_config: Dict,
        timestamp: str,
        user_id: str,
        tools: List[Dict],
        agents: List[AgentBaseModel],
    ):
        self.type = type
        self.config = config
        self.groupchat_config = groupchat_config
        self.timestamp = timestamp
        self.user_id = user_id
        self.tools = tools
        self.agents = agents

    def to_dict(self):
        return {
            "type": self.type,
            "config": self.config,
            "groupchat_config": self.groupchat_config,
            "timestamp": self.timestamp,
            "user_id": self.user_id,
            "tools": self.tools,
            "agents": [agent.to_dict() for agent in self.agents],
        }

    @classmethod
    def from_dict(cls, data: Dict):
        return cls(
            type=data["type"],
            config=data["config"],
            groupchat_config=data["groupchat_config"],
            timestamp=data["timestamp"],
            user_id=data["user_id"],
            tools=data["tools"],
            agents=[AgentBaseModel.from_dict(agent) for agent in data.get("agents", [])],
        )

class WorkflowBaseModel:
    def __init__(
        self,
        name: str,
        description: str,
        agents: List[AgentBaseModel],
        sender: Sender,
        receiver: Receiver,
        type: str,
        user_id: str,
        timestamp: str,
        summary_method: str,
        settings: Dict = None,
        groupchat_config: Dict = None,
        id: Optional[int] = None,
        created_at: Optional[str] = None,
        updated_at: Optional[str] = None,
    ):
        self.id = id
        self.name = name
        self.description = description
        self.agents = agents
        self.sender = sender
        self.receiver = receiver
        self.type = type
        self.user_id = user_id
        self.timestamp = timestamp
        self.summary_method = summary_method
        self.settings = settings or {}
        self.groupchat_config = groupchat_config or {}
        self.created_at = created_at or datetime.now().isoformat()
        self.updated_at = updated_at

    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "description": self.description,
            "agents": [agent.to_dict() for agent in self.agents],
            "sender": self.sender.to_dict(),
            "receiver": self.receiver.to_dict(),
            "type": self.type,
            "user_id": self.user_id,
            "timestamp": self.timestamp,
            "summary_method": self.summary_method,
            "settings": self.settings,
            "groupchat_config": self.groupchat_config,
            "created_at": self.created_at,
            "updated_at": self.updated_at,
        }

    @classmethod
    def create_workflow(cls, workflow_name: str) -> "WorkflowBaseModel":
        if DEBUG:
            print(f"Creating workflow {workflow_name}")
        workflow = cls(
            name=workflow_name,
            description="This workflow is used for general purpose tasks.",
            agents=[],
            sender=Sender(
                type="userproxy",
                config={
                    "name": "userproxy",
                    "llm_config": {
                        "config_list": [
                            {
                                "user_id": "default",
                                "timestamp": "2024-03-28T06:34:40.214593",
                                "model": "gpt-4o",
                                "base_url": None,
                                "api_type": None,
                                "api_version": None,
                                "description": "OpenAI model configuration"
                            }
                        ],
                        "temperature": 0.1,
                        "cache_seed": None,
                        "timeout": None,
                        "max_tokens": None,
                        "extra_body": None
                    },
                    "human_input_mode": "NEVER",
                    "max_consecutive_auto_reply": 30,
                    "system_message": "You are a helpful assistant.",
                    "is_termination_msg": None,
                    "code_execution_config": {
                        "work_dir": None,
                        "use_docker": False
                    },
                    "default_auto_reply": "TERMINATE",
                    "description": "A user proxy agent that executes code."
                },
                timestamp="2024-03-28T06:34:40.214593",
                user_id="user",
                tools=[
                    {
                        "title": "fetch_web_content",
                        "content": "...",  # Omitted for brevity
                        "file_name": "fetch_web_content.json",
                        "description": None,
                        "timestamp": "2024-05-14T08:19:12.425322",
                        "user_id": "default"
                    }
                ]
            ),
            receiver=Receiver(
                type="assistant",
                config={
                    "name": "primary_assistant",
                    "llm_config": {
                        "config_list": [
                            {
                                "user_id": "default",
                                "timestamp": "2024-05-14T08:19:12.425322",
                                "model": "gpt-4o",
                                "base_url": None,
                                "api_type": None,
                                "api_version": None,
                                "description": "OpenAI model configuration"
                            }
                        ],
                        "temperature": 0.1,
                        "cache_seed": None,
                        "timeout": None,
                        "max_tokens": None,
                        "extra_body": None
                    },
                    "human_input_mode": "NEVER",
                    "max_consecutive_auto_reply": 30,
                    "system_message": "...",  # Omitted for brevity
                    "is_termination_msg": None,
                    "code_execution_config": None,
                    "default_auto_reply": "",
                    "description": "A primary assistant agent that writes plans and code to solve tasks. booger"
                },
                groupchat_config={},
                timestamp=datetime.now().isoformat(),
                user_id="default",
                tools=[
                    {
                        "title": "fetch_web_content",
                        "content": "...",  # Omitted for brevity
                        "file_name": "fetch_web_content.json",
                        "description": None,
                        "timestamp": "2024-05-14T08:19:12.425322",
                        "user_id": "default"
                    }
                ],
                agents=[]
            ),
            type="twoagents",
            user_id="user",
            timestamp=datetime.now().isoformat(),
            summary_method="last"
        )

        # Create a YAML file for the workflow with the default values
        workflow_data = workflow.to_dict()
        with open(f"workflows/{workflow_name}.yaml", "w") as file:
            yaml.dump(workflow_data, file)

        return workflow
    
    @classmethod
    def from_dict(cls, data: Dict):
        sender = Sender.from_dict(data["sender"])
        receiver = Receiver.from_dict(data["receiver"])
        return cls(
            id=data.get("id"),
            name=data["name"],
            description=data["description"],
            agents=[AgentBaseModel.from_dict(agent) for agent in data.get("agents", [])],
            sender=sender,
            receiver=receiver,
            type=data["type"],
            user_id=data["user_id"],
            timestamp=data["timestamp"],
            summary_method=data["summary_method"],
            settings=data.get("settings", {}),
            groupchat_config=data.get("groupchat_config", {}),
            created_at=data.get("created_at"),
            updated_at=data.get("updated_at"),
        )
    

    @classmethod
    def get_workflow(cls, workflow_name: str) -> "WorkflowBaseModel":
        if DEBUG:
            print(f"Loading workflow: {workflow_name}")
        file_path = f"workflows/{workflow_name}.yaml"
        if os.path.exists(file_path):
            with open(file_path, "r") as file:
                workflow_data = yaml.safe_load(file)
                return cls.from_dict(workflow_data)
        else:
            raise FileNotFoundError(f"Workflow file not found: {file_path}")
    

    @staticmethod
    def load_workflows() -> List[str]:
        if DEBUG:
            print("Loading workflows")
        project_names = []
        for file in os.listdir("workflows"):
            if file.endswith(".yaml"):
                project_name = file[:-5]  # Remove the ".yaml" extension
                project_names.append(project_name)
        return project_names
```

# configs\config.py

```python
# config.py

import os

# Get user home directory
home_dir = os.path.expanduser("~")
default_db_path = f'{home_dir}/.autogenstudio/database.sqlite'

# Debug
DEFAULT_DEBUG = False

# Default configurations
DEFAULT_LLM_PROVIDER = "groq"
DEFAULT_GROQ_API_URL = "https://api.groq.com/openai/v1/chat/completions"
DEFAULT_LMSTUDIO_API_URL = "http://localhost:1234/v1/chat/completions"
DEFAULT_OLLAMA_API_URL = "http://127.0.0.1:11434/api/generate"
DEFAULT_OPENAI_API_KEY = None
DEFAULT_OPENAI_API_URL = "https://api.openai.com/v1/chat/completions"

# Try to import user-specific configurations from config_local.py
try:
    from config_local import *
except ImportError:
    pass

# Set the configurations using the user-specific values if available, otherwise use the defaults
DEBUG = locals().get('DEBUG', DEFAULT_DEBUG)

LLM_PROVIDER = locals().get('LLM_PROVIDER', DEFAULT_LLM_PROVIDER)
GROQ_API_URL = locals().get('GROQ_API_URL', DEFAULT_GROQ_API_URL)
LMSTUDIO_API_URL = locals().get('LMSTUDIO_API_URL', DEFAULT_LMSTUDIO_API_URL)
OLLAMA_API_URL = locals().get('OLLAMA_API_URL', DEFAULT_OLLAMA_API_URL)
OPENAI_API_KEY = locals().get('OPENAI_API_KEY', DEFAULT_OPENAI_API_KEY)
OPENAI_API_URL = locals().get('OPENAI_API_URL', DEFAULT_OPENAI_API_URL)

API_KEY_NAMES = {
    "groq": "GROQ_API_KEY",
    "lmstudio": None,
    "ollama": None,
    "openai": "OPENAI_API_KEY",
    # Add other LLM providers and their respective API key names here
}

# Retry settings
MAX_RETRIES = 3
RETRY_DELAY = 2  # in seconds
RETRY_TOKEN_LIMIT = 5000

# Model configurations
if LLM_PROVIDER == "groq":
    API_URL = GROQ_API_URL
    MODEL_TOKEN_LIMITS = {
        'mixtral-8x7b-32768': 32768,
        'llama3-70b-8192': 8192,
        'llama3-8b-8192': 8192,
        'gemma-7b-it': 8192,
    }
elif LLM_PROVIDER == "lmstudio":
    API_URL = LMSTUDIO_API_URL
    MODEL_TOKEN_LIMITS = {
        'instructlab/granite-7b-lab-GGUF': 2048,
        'MaziyarPanahi/Codestral-22B-v0.1-GGUF': 32768,
    } 
elif LLM_PROVIDER == "openai":
    API_URL = OPENAI_API_URL
    MODEL_TOKEN_LIMITS = {
        'gpt-4o': 4096,
    }
elif LLM_PROVIDER == "ollama":
    API_URL = OLLAMA_API_URL
    MODEL_TOKEN_LIMITS = {
        'llama3': 8192,
    }   
else:
    MODEL_TOKEN_LIMITS = {}

    
# Database path
# FRAMEWORK_DB_PATH="/path/to/custom/database.sqlite"
FRAMEWORK_DB_PATH = os.environ.get('FRAMEWORK_DB_PATH', default_db_path)

MODEL_CHOICES = {
    'default': None,
    'gemma-7b-it': 8192,
    'gpt-4o': 4096,
    'instructlab/granite-7b-lab-GGUF': 2048,
    'MaziyarPanahi/Codestral-22B-v0.1-GGUF': 32768,
    'llama3': 8192,
    'llama3-70b-8192': 8192,
    'llama3-8b-8192': 8192,
    'mixtral-8x7b-32768': 32768
}
```

# configs\config_local.py

```python
# config_local.py

# User-specific configurations

LLM_PROVIDER = "Groq_Provider"
DEFAULT_MODEL = "llama3-8b-8192"
GROQ_API_URL = "https://api.groq.com/openai/v1/chat/completions"
LMSTUDIO_API_URL = "http://localhost:1234/v1/chat/completions"
OLLAMA_API_URL = "http://127.0.0.1:11434/api/generate"
# OPENAI_API_KEY = "your_openai_api_key"
OPENAI_API_URL = "https://api.openai.com/v1/chat/completions"

DEBUG = True
```

# event_handlers\event_handlers_agent.py

```python
# event_handlers_agent.py

import importlib
import re
import streamlit as st
import yaml

from datetime import datetime
from base_models.agent_base_model import AgentBaseModel
from configs.config_local import DEBUG


def handle_agent_close():
    if DEBUG:
        print("handle_agent_close()")
    st.session_state.current_agent = None
    st.session_state.agent_dropdown = "Select..."
    st.rerun()
    

def handle_ai_agent_creation():
    if DEBUG:
        print("handle_ai_agent_creation()")
    agent_creation_input = st.session_state.agent_creation_input.strip()
    if agent_creation_input:
        # Load the generate_agent_prompt from the file
        with open("prompts/generate_agent_prompt.yaml", "r") as file:
            prompt_data = yaml.safe_load(file)
            if prompt_data is not None and "generate_agent_prompt" in prompt_data:
                generate_agent_prompt = prompt_data["generate_agent_prompt"]
            else:
                st.error("Failed to load the agent prompt.")
                return

        # Combine the generate_agent_prompt with the user input
        prompt = f"{generate_agent_prompt}\n\nRephrased agent request: {agent_creation_input}"

        # Dynamically import the provider class based on the selected provider name
        provider_module = importlib.import_module(f"providers.{st.session_state.default_provider.lower()}")
        provider_class = getattr(provider_module, st.session_state.default_provider)
        provider = provider_class(api_url="", api_key=st.session_state.default_provider_key)
        model = st.session_state.selected_model

        try:
            response = provider.send_request({"model": model, "messages": [{"role": "user", "content": prompt}]})
            agent_data = provider.process_response(response)["choices"][0]["message"]["content"]
            if DEBUG:
                print(f"Generated agent data:\n{agent_data}")   

            agent_name_match = re.search(r"# (\w+)\.py", agent_data)
            if agent_name_match:
                agent_name = agent_name_match.group(1)
                agent_data_dict = {
                    "name": agent_name,
                    "config": {"name": agent_name},
                    "tools": [],
                    "code": agent_data
                }
                agent = AgentBaseModel.create_agent(agent_name, agent_data_dict)
                st.session_state.current_agent = agent
                st.session_state.agent_dropdown = agent_name
                st.success(f"Agent '{agent_name}' created successfully!")
            else:
                st.error("Failed to extract the agent name from the generated data.")
        except Exception as e:
            st.error(f"Error generating the agent: {str(e)}")


def handle_agent_property_change():
    if DEBUG:
        print("handle_agent_property_change()")
    agent = st.session_state.current_agent
    if agent:
        agent.name = st.session_state[f"agent_name_{agent.name}"]
        agent.description = st.session_state[f"agent_description_{agent.name}"]
        agent.role = st.session_state[f"agent_role_{agent.name}"]
        agent.goal = st.session_state[f"agent_goal_{agent.name}"]
        agent.backstory = st.session_state[f"agent_backstory_{agent.name}"]

        agent_data = agent.to_dict()
        agent_name = agent.name
        with open(f"agents/{agent_name}.yaml", "w") as file:
            yaml.dump(agent_data, file)


def handle_agent_selection():
    if DEBUG:
        print("handle_agent_selection()")
    selected_agent = st.session_state.agent_dropdown
    if selected_agent == "Select...":
        return
    if selected_agent == "Create manually...":
        # Handle manual agent creation
        agent_name = st.session_state.agent_name_input.strip()
        if agent_name:
            agent_data = {
                "name": agent_name,
                "description": "",
                "role": "",
                "goal": "",
                "backstory": "",
                "tools": [],
                "config": {},
                "timestamp": datetime.now().isoformat(),
                "user_id": "default"
            }
            agent = AgentBaseModel.from_dict(agent_data)
            AgentBaseModel.create_agent(agent_name, agent)
            st.session_state.current_agent = agent
            st.session_state.agent_dropdown = agent_name
    elif selected_agent == "Create with AI...":
        # Clear the current agent selection
        st.session_state.current_agent = None
    else:
        # Load the selected agent
        agent = AgentBaseModel.get_agent(selected_agent)
        st.session_state.current_agent = agent


def handle_agent_name_change():
    if DEBUG:
        print("handle_agent_name_change()")
    new_agent_name = st.session_state.agent_name_edit.strip()
    if new_agent_name:
        st.session_state.current_agent.name = new_agent_name
        update_agent()


def update_agent():
    if DEBUG:
        print("update_agent()")
    st.session_state.current_agent.updated_at = datetime.now().isoformat()
    agent_name = st.session_state.current_agent.name
    agent_data = st.session_state.current_agent.to_dict()
    with open(f"agents/{agent_name}.yaml", "w") as file:
        yaml.dump(agent_data, file)
```

# event_handlers\event_handlers_project.py

```python
# event_handlers.py

import os
import streamlit as st
import yaml

from datetime import datetime
from base_models.project_base_model import ProjectBaseModel
from base_models.workflow_base_model import WorkflowBaseModel
from configs.config_local import DEBUG
from event_handlers.event_handlers_workflow import handle_workflow_close
from event_handlers.event_handlers_shared import update_project

# def handle_project_attachments_change():
#     new_project_attachments = st.session_state.current_project.attachments.strip()
#     if new_project_attachments:
#         attachments = [attachment.strip() for attachment in new_project_attachments.split(",")]
#         st.session_state.current_project.attachments = attachments
#         update()


def handle_project_collaborators_change():
    if DEBUG:
        print("called handle_project_collaborators_change()")
    new_project_collaborators = st.session_state.project_collaborators.strip()
    if new_project_collaborators:
        collaborators = [collaborator.strip() for collaborator in new_project_collaborators.split(",")]
        st.session_state.current_project.collaborators = collaborators
        update_project()


def handle_project_close():
    if DEBUG:
        print("handle_project_close()")
    st.session_state.current_project = None
    st.session_state.project_dropdown = "Select..."
    
    # Close the current workflow
    handle_workflow_close()
    
    st.rerun()


def handle_project_delete():
    if DEBUG:
        print("handle_project_delete()")
    project_name = st.session_state.current_project.name
    project_file = f"projects/{project_name}.yaml"
    
    if os.path.exists(project_file):
        if st.sidebar.button(f"Delete Project '{project_name}'", key=f"delete_project_{project_name}"):
            st.sidebar.warning(f"Are you sure you want to delete the project '{project_name}'?")
            if st.sidebar.button("Confirm Delete", key=f"confirm_delete_project_{project_name}"):
                os.remove(project_file)
                st.session_state.current_project = None
                st.session_state.project_dropdown = "Select..."
                st.success(f"Project '{project_name}' has been deleted.")
                st.rerun()
    else:
        st.error(f"Project file '{project_file}' not found.")


def handle_project_description_change():
    if DEBUG:
        print("handle_project_description_change")
    new_project_description = st.session_state.project_description.strip()
    if new_project_description:
        st.session_state.current_project.description = new_project_description
        update_project()


def handle_project_due_date_change():
    if DEBUG:
        print("called handle_project_due_date_change()")
    new_project_due_date = st.session_state.project_due_date
    if new_project_due_date:
        st.session_state.current_project.due_date = new_project_due_date.strftime("%Y-%m-%d")
        update_project()


def handle_project_name_change():
    if DEBUG:
        print("called handle_project_name_change()")
    new_project_name = st.session_state.project_name_edit.strip()
    if new_project_name:
        st.session_state.current_project.name = new_project_name
        update_project()


def handle_project_notes_change():
    if DEBUG:
        print("called handle_project_notes_change()")
    new_project_notes = st.session_state.project_notes.strip()
    if new_project_notes:
        st.session_state.current_project.notes = new_project_notes
        update_project()


def handle_project_selection():
    if DEBUG:
        print("called handle_project_selection()")
    selected_project = st.session_state.project_dropdown
    if selected_project == "Select...":
        return
    if selected_project == "Create...":
        project_name = st.session_state.project_name_input.strip()
        if project_name:
            project = ProjectBaseModel(name=project_name)
            st.session_state.current_project = project
            st.session_state.project_dropdown = project_name
            ProjectBaseModel.create_project(st.session_state.current_project.name)
            
            # Close the current workflow
            handle_workflow_close()
    else:
        # Load the selected project
        project = ProjectBaseModel.get_project(selected_project)
        st.session_state.current_project = project

        # Check if the specified workflow exists and load it
        if project.workflows:
            workflow_name = project.workflows[0]
            try:
                workflow = WorkflowBaseModel.get_workflow(workflow_name)
                st.session_state.current_workflow = workflow
                st.session_state.workflow_dropdown = workflow_name
            except FileNotFoundError:
                st.warning(f"Workflow '{workflow_name}' not found for project '{project.name}'.")


def handle_project_status_change():
    if DEBUG:
        print("called handle_project_status_change()")
    new_project_status = st.session_state.project_status
    if new_project_status:
        st.session_state.current_project.status = new_project_status
        update_project()


def handle_project_user_id_change():
    if DEBUG:
        print("called handle_project_user_id_change()")
    new_project_user_id = st.session_state.project_user_id.strip()
    if new_project_user_id:
        st.session_state.current_project.user_id = new_project_user_id
        update_project()


def handle_prompt_change():
    if DEBUG:
        print("called handle_prompt_change()")
    new_prompt = st.session_state.prompt.strip()
    if new_prompt:
        st.session_state.current_project.prompt = new_prompt
        update_project()

```

# event_handlers\event_handlers_settings.py

```python
# event_handlers\event_handlers_settings.py

import importlib
import os
import streamlit as st

from configs.config import DEBUG
from providers.base_provider import BaseLLMProvider


def handle_default_provider_change():
    if DEBUG:
        print("handle_default_provider_change()")
    selected_provider = st.session_state.default_provider
    st.session_state.default_provider = selected_provider


def load_model_classes():
    if DEBUG:
        print("load_model_classes()")
    model_classes = []
    models_folder = "models"

    for file_name in os.listdir(models_folder):
        if file_name.endswith(".py") and file_name != "llm_base_model.py":
            module_name = file_name[:-3]  # Remove the ".py" extension
            module = importlib.import_module(f"{models_folder}.{module_name}")

            for attr_name in dir(module):
                attr = getattr(module, attr_name)
                if isinstance(attr, type) and issubclass(attr, BaseLLMProvider) and attr != BaseLLMProvider:
                    model_classes.append(attr_name)

    return model_classes


def load_provider_classes():
    if DEBUG:
        print("load_provider_classes()")
    provider_classes = []
    providers_folder = "providers"
    
    for file_name in os.listdir(providers_folder):
        if file_name.endswith(".py") and file_name != "base_provider.py":
            module_name = file_name[:-3]  # Remove the ".py" extension
            module = importlib.import_module(f"{providers_folder}.{module_name}")
            
            for attr_name in dir(module):
                attr = getattr(module, attr_name)
                if isinstance(attr, type) and issubclass(attr, BaseLLMProvider) and attr != BaseLLMProvider:
                    provider_classes.append(attr_name)
    
    return provider_classes
```

# event_handlers\event_handlers_shared.py

```python
import streamlit as st
import yaml

from configs.config_local import DEBUG
from datetime import datetime

def update_project():
    if DEBUG:
        print("called update_project()")
    st.session_state.current_project.updated_at = datetime.now().isoformat()
    project_name = st.session_state.current_project.name
    project_data = st.session_state.current_project.to_dict()
    with open(f"projects/{project_name}.yaml", "w") as file:
        yaml.dump(project_data, file)
```

# event_handlers\event_handlers_tool.py

```python
# event_handlers_tool.py

import importlib
import re
import streamlit as st
import yaml

from datetime import datetime
from base_models.tool_base_model import ToolBaseModel
from configs.config_local import DEBUG


def handle_ai_tool_creation():
    if DEBUG:
        print("handle_ai_tool_creation()")
    tool_creation_input = st.session_state.tool_creation_input.strip()
    if tool_creation_input:
        # Load the generate_tool_prompt from the file
        with open("prompts/generate_tool_prompt.yaml", "r") as file:
            prompt_data = yaml.safe_load(file)
            generate_tool_prompt = prompt_data["generate_tool_prompt"]

        # Combine the generate_tool_prompt with the user input
        prompt = f"{generate_tool_prompt}\n\nRephrased tool request: {tool_creation_input}"

        # Dynamically import the provider class based on the selected provider name
        provider_module = importlib.import_module(f"providers.{st.session_state.default_provider.lower()}")
        provider_class = getattr(provider_module, st.session_state.default_provider)
        provider = provider_class(api_url="", api_key=st.session_state.default_provider_key)
        model = st.session_state.selected_model

        try:
            response = provider.send_request({"model": model, "messages": [{"role": "user", "content": prompt}]})
            tool_code = provider.process_response(response)["choices"][0]["message"]["content"]

            # Extract the tool name from the generated code
            tool_name_match = re.search(r"# Tool filename: ([\w_]+)\.py", tool_code)
            if tool_name_match:
                tool_name = tool_name_match.group(1)
                tool_data = {
                    "name": tool_name,
                    "title": tool_name,
                    "content": tool_code,
                    "file_name": f"{tool_name}.json",
                    "description": None,
                    "timestamp": datetime.now().isoformat(),
                    "user_id": "default"
                }
                tool = ToolBaseModel.create_tool(tool_name, tool_data)
                st.session_state.current_tool = tool
                st.session_state.tool_dropdown = tool_name
                st.success(f"Tool '{tool_name}' created successfully!")
            else:
                st.error("Failed to extract the tool name from the generated code.")
        except Exception as e:
            st.error(f"Error generating the tool: {str(e)}")


def handle_tool_close():
    if DEBUG:
        print("handle_tool_close()")
    st.session_state.current_tool = None
    st.session_state.tool_dropdown = "Select..."
    st.rerun()


def handle_tool_property_change():
    if DEBUG:
        print("handle_tool_property_change()")
    tool = st.session_state.current_tool
    if tool:
        tool.name = st.session_state[f"tool_name_{tool.name}"]
        tool.title = st.session_state[f"tool_title_{tool.name}"]
        tool.description = st.session_state[f"tool_description_{tool.name}"]
        tool.file_name = st.session_state[f"tool_file_name_{tool.name}"]
        tool.content = st.session_state[f"tool_content_{tool.name}"]
        tool.user_id = st.session_state[f"tool_user_id_{tool.name}"]

        tool_data = tool.to_dict()
        tool_name = tool.name
        with open(f"tools/{tool_name}.yaml", "w") as file:
            yaml.dump(tool_data, file)


def handle_tool_selection():
    if DEBUG:
        print("handle_tool_selection()")
    selected_tool = st.session_state.tool_dropdown
    if selected_tool     == "Select...":
        return
    if selected_tool == "Create manually...":
        # Handle manual tool creation
        tool_name = st.session_state.tool_name_input.strip()
        if tool_name:
            tool_data = {
                "name": tool_name,
                "title": tool_name,
                "content": "",
                "file_name": f"{tool_name}.json",
                "description": None,
                "timestamp": datetime.now().isoformat(),
                "user_id": "default"
            }
            tool = ToolBaseModel.create_tool(tool_name, tool_data)
            st.session_state.current_tool = tool
            st.session_state.tool_dropdown = tool_name
    elif selected_tool == "Create with AI...":
        # Clear the current tool selection
        st.session_state.current_tool = None
    else:
        # Load the selected tool
        tool = ToolBaseModel.get_tool(selected_tool)
        st.session_state.current_tool = tool


def handle_tool_name_change():
    if DEBUG:
        print("handle_tool_name_change()")
    new_tool_name = st.session_state.tool_name_edit.strip()
    if new_tool_name:
        st.session_state.current_tool.name = new_tool_name
        update_tool()


def update_tool():
    if DEBUG:
        print("update_tool()")
    st.session_state.current_tool.updated_at = datetime.now().isoformat()
    tool_name = st.session_state.current_tool.name
    tool_data = st.session_state.current_tool.to_dict()
    with open(f"tools/{tool_name}.yaml", "w") as file:
        yaml.dump(tool_data, file)
```

# event_handlers\event_handlers_workflow.py

```python
# event_handlers_workflow.py

import os
import streamlit as st
import yaml

from datetime import datetime
from base_models.workflow_base_model import WorkflowBaseModel, Sender, Receiver
from configs.config_local import DEBUG
from event_handlers.event_handlers_shared import update_project


def handle_workflow_close():
    if DEBUG:
        print("called handle_workflow_close()")
    st.session_state.current_workflow = None
    st.session_state.workflow_dropdown = "Select..."
    st.rerun()


def handle_workflow_delete(workflow_file):
    if DEBUG:
        print(f"called handle_workflow_delete({workflow_file})")
    os.remove(workflow_file)
    st.session_state.current_workflow = None
    st.session_state.workflow_dropdown = "Select..."
    st.success(f"Workflow '{workflow_file}' has been deleted.")


def handle_workflow_description_change():
    if DEBUG:
        print("called handle_workflow_description_change()")
    new_workflow_description = st.session_state.workflow_description.strip()
    if new_workflow_description:
        st.session_state.current_workflow.description = new_workflow_description
        update_workflow()


def handle_workflow_name_change():
    if DEBUG:
        print("called handle_workflow_name_change()")
    new_workflow_name = st.session_state.workflow_name_edit.strip()
    if new_workflow_name:
        old_workflow_name = st.session_state.current_workflow.name
        st.session_state.current_workflow.name = new_workflow_name
        update_workflow()
        
        # Update the workflow name in current_project.workflows
        if st.session_state.current_project and old_workflow_name in st.session_state.current_project.workflows:
            index = st.session_state.current_project.workflows.index(old_workflow_name)
            st.session_state.current_project.workflows[index] = new_workflow_name
            update_project()


def handle_workflow_selection():
    if DEBUG:
        print("called handle_workflow_selection()")
    selected_workflow = st.session_state.workflow_dropdown
    if selected_workflow == "Select...":
        return
    if selected_workflow == "Create...":
        workflow_name = st.session_state.workflow_name_input.strip()
        if workflow_name:
            workflow = WorkflowBaseModel(
                name=workflow_name,
                description="This workflow is used for general purpose tasks.",
                agents=[],
                sender=Sender(
                    type="userproxy",
                    config={
                        "name": "userproxy",
                        "llm_config": {
                            "config_list": [
                                {
                                    "user_id": "default",
                                    "timestamp": "2024-03-28T06:34:40.214593",
                                    "model": "gpt-4o",
                                    "base_url": None,
                                    "api_type": None,
                                    "api_version": None,
                                    "description": "OpenAI model configuration"
                                }
                            ],
                            "temperature": 0.1,
                            "cache_seed": None,
                            "timeout": None,
                            "max_tokens": None,
                            "extra_body": None
                        },
                        "human_input_mode": "NEVER",
                        "max_consecutive_auto_reply": 30,
                        "system_message": "You are a helpful assistant.",
                        "is_termination_msg": None,
                        "code_execution_config": {
                            "work_dir": None,
                            "use_docker": False
                        },
                        "default_auto_reply": "TERMINATE",
                        "description": "A user proxy agent that executes code."
                    },
                    timestamp="2024-03-28T06:34:40.214593",
                    user_id="user",
                    tools=[
                        {
                            "title": "fetch_web_content",
                            "content": "from typing import Optional\nimport requests\nimport collections\ncollections.Callable = collections.abc.Callable\nfrom bs4 import BeautifulSoup\n\ndef fetch_web_content(url: str) -> Optional[str]:\n    \"\"\"\n    Fetches the text content from a website.\n\n    Args:\n        url (str): The URL of the website.\n\n    Returns:\n        Optional[str]: The content of the website.\n    \"\"\"\n    try:\n        # Send a GET request to the URL\n        response = requests.get(url)\n\n        # Check for successful access to the webpage\n        if response.status_code == 200:\n            # Parse the HTML content of the page using BeautifulSoup\n            soup = BeautifulSoup(response.text, \"html.parser\")\n\n            # Extract the content of the <body> tag\n            body_content = soup.body\n\n            if body_content:\n                # Return all the text in the body tag, stripping leading/trailing whitespaces\n                return \" \".join(body_content.get_text(strip=True).split())\n            else:\n                # Return None if the <body> tag is not found\n                return None\n        else:\n            # Return None if the status code isn't 200 (success)\n            return None\n    except requests.RequestException:\n        # Return None if any request-related exception is caught\n        return None",
                            "file_name": "fetch_web_content.json",
                            "description": None,
                            "timestamp": "2024-05-14T08:19:12.425322",
                            "user_id": "default"
                        }
                    ]
                ),
                receiver=Receiver(
                    type="assistant",
                    config={
                        "name": "primary_assistant",
                        "llm_config": {
                            "config_list": [
                                {
                                    "user_id": "default",
                                    "timestamp": "2024-05-14T08:19:12.425322",
                                    "model": "gpt-4o",
                                    "base_url": None,
                                    "api_type": None,
                                    "api_version": None,
                                    "description": "OpenAI model configuration"
                                }
                            ],
                            "temperature": 0.1,
                            "cache_seed": None,
                            "timeout": None,
                            "max_tokens": None,
                            "extra_body": None
                        },
                        "human_input_mode": "NEVER",
                        "max_consecutive_auto_reply": 30,
                        "system_message": "You are a helpful AI assistant. Solve tasks using your coding and language skills. In the following cases, suggest python code (in a python coding block) or shell script (in a sh coding block) for the user to execute. 1. When you need to collect info, use the code to output the info you need, for example, browse or search the web, download/read a file, print the content of a webpage or a file, get the current date/time, check the operating system. After sufficient info is printed and the task is ready to be solved based on your language skill, you can solve the task by yourself. 2. When you need to perform some task with code, use the code to perform the task and output the result. Finish the task smartly. Solve the task step by step if you need to. If a plan is not provided, explain your plan first. Be clear which step uses code, and which step uses your language skill. When using code, you must indicate the script type in the code block. The user cannot provide any other feedback or perform any other action beyond executing the code you suggest. The user can't modify your code. So do not suggest incomplete code which requires users to modify. Don't use a code block if it's not intended to be executed by the user. If you want the user to save the code in a file before executing it, put # filename: <filename> inside the code block as the first line. Don't include multiple code blocks in one response. Do not ask users to copy and paste the result. Instead, use 'print' function for the output when relevant. Check the execution result returned by the user. If the result indicates there is an error, fix the error and output the code again. Suggest the full code instead of partial code or code changes. If the error can't be fixed or if the task is not solved even after the code is executed successfully, analyze the problem, revisit your assumption, collect additional info you need, and think of a different approach to try. When you find an answer, verify the answer carefully. Include verifiable evidence in your response if possible. Reply 'TERMINATE' in the end when everything is done.",
                        "is_termination_msg": None,
                        "code_execution_config": None,
                        "default_auto_reply": "",
                        "description": "A primary assistant agent that writes plans and code to solve tasks. booger"
                    },
                    groupchat_config={},
                    timestamp=datetime.now().isoformat(),
                    user_id="default",
                    tools=[
                        {
                            "title": "fetch_web_content",
                            "content": "from typing import Optional\nimport requests\nimport collections\ncollections.Callable = collections.abc.Callable\nfrom bs4 import BeautifulSoup\n\ndef fetch_web_content(url: str) -> Optional[str]:\n    \"\"\"\n    Fetches the text content from a website.\n\n    Args:\n        url (str): The URL of the website.\n\n    Returns:\n        Optional[str]: The content of the website.\n    \"\"\"\n    try:\n        # Send a GET request to the URL\n        response = requests.get(url)\n\n        # Check for successful access to the webpage\n        if response.status_code == 200:\n            # Parse the HTML content of the page using BeautifulSoup\n            soup = BeautifulSoup(response.text, \"html.parser\")\n\n            # Extract the content of the <body> tag\n            body_content = soup.body\n\n            if body_content:\n                # Return all the text in the body tag, stripping leading/trailing whitespaces\n                return \" \".join(body_content.get_text(strip=True).split())\n            else:\n                # Return None if the <body> tag is not found\n                return None\n        else:\n            # Return None if the status code isn't 200 (success)\n            return None\n    except requests.RequestException:\n        # Return None if any request-related exception is caught\n        return None",
                            "file_name": "fetch_web_content.json",
                            "description": None,
                            "timestamp": "2024-05-14T08:19:12.425322",
                            "user_id": "default"
                        }
                    ],
                    agents=[]
                ),
                type="twoagents",
                user_id="user",
                timestamp=datetime.now().isoformat(),
                summary_method="last"
            )
            st.session_state.current_workflow = workflow
            st.session_state.workflow_dropdown = workflow_name
            WorkflowBaseModel.create_workflow(st.session_state.current_workflow.name)

            # Add the created workflow's name to current_project.workflows
            if st.session_state.current_project:
                st.session_state.current_project.workflows = [workflow_name] + st.session_state.current_project.workflows[1:]
                update_project()
    else:
        print ("Selected workflow: ", selected_workflow)
        workflow = WorkflowBaseModel.get_workflow(selected_workflow)
        st.session_state.current_workflow = workflow

        # Update current_project.workflows to reflect the selected workflow
        if st.session_state.current_project:
            st.session_state.current_project.workflows = [selected_workflow] + [wf for wf in st.session_state.current_project.workflows if wf != selected_workflow]
            update_project()



def handle_workflow_type_change():
    if DEBUG:
        print("called handle_workflow_type_change()")
    new_workflow_type = st.session_state.workflow_type.strip()
    if new_workflow_type:
        st.session_state.current_workflow.type = new_workflow_type
        update_workflow()


def handle_workflow_summary_method_change():
    if DEBUG:
        print("called handle_workflow_summary_method_change()")
    new_workflow_summary_method = st.session_state.workflow_summary_method.strip()
    if new_workflow_summary_method:
        st.session_state.current_workflow.summary_method = new_workflow_summary_method
        update_workflow()


def update_workflow():
    if DEBUG:
        print("called update_workflow()")
    st.session_state.current_workflow.updated_at = datetime.now().isoformat()
    workflow_name = st.session_state.current_workflow.name
    workflow_data = st.session_state.current_workflow.to_dict()
    with open(f"workflows/{workflow_name}.yaml", "w") as file:
        yaml.dump(workflow_data, file)
```

# providers\base_provider.py

```python

from abc import ABC, abstractmethod

class BaseLLMProvider(ABC):
    @abstractmethod
    def send_request(self, data):
        pass

    @abstractmethod
    def process_response(self, response):
        pass
    
```

# providers\fireworks_provider.py

```python

import json
import requests

from providers.base_provider import BaseLLMProvider
from utils.auth_utils import get_api_key


class FireworksProvider(BaseLLMProvider):
    def __init__(self, api_url):
        self.api_key = get_api_key()
        self.api_url = api_url


    def process_response(self, response):
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Request failed with status code {response.status_code}")


    def send_request(self, data):
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
        }
        # Ensure data is a JSON string
        if isinstance(data, dict):
            json_data = json.dumps(data)
        else:
            json_data = data
        response = requests.post(self.api_url, data=json_data, headers=headers)
        return response
    
```

# providers\groq_provider.py

```python
# groq_provider.py

import json
import os
import requests
import streamlit as st

from configs.config_local import DEBUG
from providers.base_provider import BaseLLMProvider

class Groq_Provider(BaseLLMProvider):
    def __init__(self, api_url, api_key):
        self.api_key = api_key
        if api_url:
            self.api_url = api_url
        else:
            self.api_url = "https://api.groq.com/openai/v1/chat/completions"


    def get_available_models(self):
        if DEBUG:
            print ("GROQ: get_available_models")
            print (f"KEY: {self.api_key}")
        response = requests.get("https://api.groq.com/openai/v1/models", headers={
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
        })
        if response.status_code == 200:
            models = response.json().get("data", [])
            return [model["id"] for model in models]
        else:
            raise Exception(f"Failed to retrieve models: {response.status_code}")


    def process_response(self, response):
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Request failed with status code {response.status_code}")


    def send_request(self, data):
        # Check for API key in environment variable
        api_key = os.environ.get("GROQ_API_KEY")
        
        # If not found in environment variable, check session state
        if not api_key:
            api_key = st.session_state.get("default_provider_key")
        
        # If not found in session state, check global variable
        if not api_key:
            api_key = globals().get("GROQ_API_KEY")
        
        # If no API key is found, raise an exception
        if not api_key:
            raise Exception("No Groq API key found. Please provide an API key.")
        
        headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
        }
        # Ensure data is a JSON string
        if isinstance(data, dict):
            json_data = json.dumps(data)
        else:
            json_data = data
        response = requests.post(self.api_url, data=json_data, headers=headers)
        return response
    
```

# providers\lmstudio_provider.py

```python
# lmstudio_provider.py

import json
import requests
import streamlit as st

from providers.base_provider import BaseLLMProvider

class LmstudioProvider(BaseLLMProvider):
    def __init__(self, api_url, api_key=None):
        self.api_url = "http://localhost:1234/v1/chat/completions"

    def process_response(self, response):
        if response.status_code == 200:
            response_data = response.json()
            if "choices" in response_data:
                content = response_data["choices"][0]["message"]["content"]
                return {
                    "choices": [
                        {
                            "message": {
                                "content": content.strip()
                            }
                        }
                    ]
                }
            else:
                raise Exception("Unexpected response format. 'choices' field missing.")
        else:
            raise Exception(f"Request failed with status code {response.status_code}")


    def send_request(self, data):
        headers = {
            "Content-Type": "application/json",
        }

        # Construct the request data in the format expected by the LM Studio API
        lm_studio_request_data = {
            "model": data["model"],
            "messages": data["messages"],
            "temperature": st.session_state.temperature,
            "max_tokens": data.get("max_tokens", 2048),
            "stop": data.get("stop", "TERMINATE"),
        }

        # Ensure data is a JSON string
        if isinstance(lm_studio_request_data, dict):
            json_data = json.dumps(lm_studio_request_data)
        else:
            json_data = lm_studio_request_data

        response = requests.post(self.api_url, data=json_data, headers=headers)
        return response
    
```

# providers\ollama_provider.py

```python
import json
import requests
import streamlit as st

from providers.base_provider import BaseLLMProvider

class OllamaProvider(BaseLLMProvider):
    def __init__(self, api_url, api_key=None):
        self.api_url = "http://127.0.0.1:11434/api/generate"


    def process_response(self, response):
        if response.status_code == 200:
            response_data = response.json()
            if "response" in response_data:
                content = response_data["response"].strip()
                if content:
                    return {
                        "choices": [
                            {
                                "message": {
                                    "content": content
                                }
                            }
                        ]
                    }
                else:
                    raise Exception("Empty response received from the Ollama API.")
            else:
                raise Exception("Unexpected response format. 'response' field missing.")
        else:
            raise Exception(f"Request failed with status code {response.status_code}")


    def send_request(self, data):
        headers = {
            "Content-Type": "application/json",
        }
        # Construct the request data in the format expected by the Ollama API
        ollama_request_data = {
            "model": data["model"],
            "prompt": data["messages"][0]["content"],
            "temperature": st.session_state.temperature,
            "max_tokens": data.get("max_tokens", 2048),
            "stop": data.get("stop", "TERMINATE"),
            "stream": False,
        }
        # Ensure data is a JSON string
        if isinstance(ollama_request_data, dict):
            json_data = json.dumps(ollama_request_data)
        else:
            json_data = ollama_request_data
        response = requests.post(self.api_url, data=json_data, headers=headers)
        return response
```

# providers\openai_provider.py

```python
# openai_provider.py

import json
import os
import requests
import streamlit as st

from configs.config_local import DEBUG
from providers.base_provider import BaseLLMProvider

class Openai_Provider(BaseLLMProvider):
    def __init__(self, api_url, api_key):
        self.api_key = os.environ.get("OPENAI_API_KEY")
        self.api_url = "https://api.openai.com/v1/chat/completions"


    def get_available_models(self):
        if DEBUG:
            print ("GROQ: get_available_models")
            print (f"KEY: {self.api_key}")
        response = requests.get("https://api.openai.com/v1/models", headers={
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
        })
        if response.status_code == 200:
            models = response.json().get("data", [])
            return [model["id"] for model in models]
        else:
            raise Exception(f"Failed to retrieve models: {response.status_code}")
        

    def process_response(self, response):
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Request failed with status code {response.status_code}")


    def send_request(self, data):
        print("self.api_url: ", self.api_url)
        
        # Check for API key in environment variable
        api_key = os.environ.get("OPENAI_API_KEY")
        
        # If not found in environment variable, check session state
        if not api_key:
            api_key = st.session_state.get("default_provider_key")
        
        # If not found in session state, check global variable
        if not api_key:
            api_key = globals().get("OPENAI_API_KEY")
        
        # If no API key is found, raise an exception
        if not api_key:
            raise Exception("No OpenAI API key found. Please provide an API key.")
        
        headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
        }
        
        # Ensure data is a JSON string
        if isinstance(data, dict):
            json_data = json.dumps(data)
        else:
            json_data = data
        
        response = requests.post(self.api_url, data=json_data, headers=headers)
        print("response.status_code: ", response.status_code)
        print("response.text: ", response.text)
        return response
    
```

# utils\auth_utils.py

```python

import os
import streamlit as st

from configs.config_local import LLM_PROVIDER


def get_api_key():
    api_key_env_var = f"{LLM_PROVIDER.upper()}_API_KEY"
    api_key = os.environ.get(api_key_env_var)
    if api_key is None:
        api_key = st.session_state.get(api_key_env_var)
    return api_key


def get_api_url():
    api_url_env_var = f"{LLM_PROVIDER.upper()}_API_URL"
    api_url = os.environ.get(api_url_env_var)
    if api_url is None:
        api_url = globals().get(api_url_env_var)
        if api_url is None:
            if api_url_env_var not in st.session_state:
                api_url = st.text_input(f"Enter the {LLM_PROVIDER.upper()} API URL:", type="password", key=f"{LLM_PROVIDER}_api_url_input")
                if api_url:
                    st.session_state[api_url_env_var] = api_url
                    st.success("API URL entered successfully.")
                else:
                    st.warning(f"Please enter the {LLM_PROVIDER.upper()} API URL to use the app.")
            else:
                api_url = st.session_state.get(api_url_env_var)
    return api_url

```

# utils\display_util.py

```python
# display_util.py

import importlib
import os
import streamlit as st

from base_models.agent_base_model import AgentBaseModel
from base_models.tool_base_model import ToolBaseModel
from base_models.project_base_model import (
    ProjectBaseModel, ProjectPriority, ProjectStatus,
)
from base_models.workflow_base_model import WorkflowBaseModel
from configs.config_local import DEBUG
from datetime import datetime
from event_handlers.event_handlers_agent import (
    handle_agent_close, handle_agent_selection, handle_ai_agent_creation, handle_agent_property_change
    )
from event_handlers.event_handlers_project import (
    handle_project_collaborators_change, handle_project_close, 
    handle_project_description_change, handle_project_due_date_change, 
    handle_project_name_change, handle_project_notes_change, 
    handle_project_selection, handle_project_status_change, handle_project_user_id_change, 
    handle_prompt_change
)
from event_handlers.event_handlers_settings import handle_default_provider_change, load_provider_classes
from event_handlers.event_handlers_tool import (
    handle_ai_tool_creation, handle_tool_close, handle_tool_property_change, handle_tool_selection, 
)
from event_handlers.event_handlers_workflow import (
    handle_workflow_close, handle_workflow_description_change, 
    handle_workflow_name_change, handle_workflow_selection, handle_workflow_summary_method_change,
    handle_workflow_type_change
)   
from providers.groq_provider import Groq_Provider
from providers.openai_provider import Openai_Provider


def display_main():
    if DEBUG:
        print("called display_main()")
    
    with open("styles.css") as f:
        st.markdown(f'<style>{f.read()}</style>', unsafe_allow_html=True)
    
    projectTab, workflowTab, agentTab, toolTab, settingsTab, debugTab = st.tabs(["Project", "Workflows", "Agents", "Tools", "Settings", "Debug"])

    with projectTab:
        col1, col2 = st.columns(2)
        with col1:
            display_project_dropdown()
                
        with col2:
            if st.session_state.current_project is not None:
                project = st.session_state.current_project
                st.write("<div class='project-properties'>PROJECT PROPERTIES</div>", unsafe_allow_html=True)
                if project.created_at:
                    created_at = datetime.fromisoformat(project.created_at).strftime("%B %d, %Y %I:%M %p")
                    st.write(f"<div class='timestamp'>Created At: {created_at}</div>", unsafe_allow_html=True)
                if project.updated_at:
                    updated_at = datetime.fromisoformat(project.updated_at).strftime("%B %d, %Y %I:%M %p")
                    st.write(f"<div class='timestamp'>Updated At: {updated_at}</div>", unsafe_allow_html=True)  

        # Display the properties of the current project
        if st.session_state.current_project is not None:
            # Display the name of the first element in the project.workflows array
            if len(st.session_state.current_project.workflows) > 0:
                workflow = st.session_state.current_project.workflows[0]
                st.write(f"<div class='workflow-name'>Workflow: {workflow}</div>", unsafe_allow_html=True)
            project.prompt = st.text_area("Prompt:", value=project.prompt, key="prompt", on_change=handle_prompt_change)
            project.description = st.text_area("Description:", value=project.description or "", key="project_description", on_change=handle_project_description_change)
            status_options = [status.value for status in ProjectStatus]
            project.status = st.selectbox("Status:", options=status_options, index=status_options.index(project.status), key="project_status", on_change=handle_project_status_change)
            
            if project.due_date:
                if isinstance(project.due_date, str):
                    due_date_value = datetime.strptime(project.due_date, "%Y-%m-%d").date()
                else:
                    due_date_value = project.due_date
            else:
                due_date_value = None
            
            # Display the date input field
            due_date = st.date_input("Due Date:", value=due_date_value, key="project_due_date")
            
            # Update the project's due date if it has changed
            if due_date != due_date_value:
                project.due_date = due_date.strftime("%Y-%m-%d") if due_date else None
                handle_project_due_date_change()
            
            priority_options = [priority.value for priority in ProjectPriority]
            project.priority = st.selectbox("Priority:", options=priority_options, index=priority_options.index(project.priority), key="project_priority", on_change=handle_project_status_change)
            st.write(f"Tags: {', '.join(project.tags)}")
            
            # Add text input field for notes
            project.notes = st.text_area("Notes:", value=project.notes or "", key="project_notes", on_change=handle_project_notes_change)
            collaborators_input = st.text_input("Collaborators:", value=", ".join(project.collaborators), key="project_collaborators", on_change=handle_project_collaborators_change)
            project.collaborators = [collaborator.strip() for collaborator in collaborators_input.split(",")]
            project.user_id = st.text_input("User ID:", value=project.user_id or "", key="project_user_id", on_change=handle_project_user_id_change)

            # st.write("Attachments:")
            # for attachment in project.attachments:
            #     st.write(f"- {attachment}")

            # new_attachment = st.text_input("Add Attachment:", key="project_new_attachment")
            # if new_attachment:
            #     project.attachments.append(new_attachment)
            #     handle_project_attachments_change()


    with workflowTab:
        workflow = st.session_state.current_workflow
        col1, col2 = st.columns(2)
        with col1:
            display_workflow_dropdown()
                        
        with col2:
            if st.session_state.current_workflow is not None:
                st.write("<div style='color:#33FFFC; font-weight:bold; text-align:right; width:100%;'>WORKFLOW PROPERTIES</div>", unsafe_allow_html=True)
                if workflow.created_at:
                    created_at = datetime.fromisoformat(workflow.created_at).strftime("%B %d, %Y %I:%M %p")
                    st.write(f"<div style='text-align:right; width:100%;'>Created At: {created_at}</div>", unsafe_allow_html=True)
                if workflow.updated_at:
                    updated_at = datetime.fromisoformat(workflow.updated_at).strftime("%B %d, %Y %I:%M %p")
                    st.write(f"<div style='text-align:right; width:100%;'>Updated At: {updated_at}</div>", unsafe_allow_html=True)


        # Display the properties of the current workflow
        if st.session_state.current_workflow is not None:
            workflow = st.session_state.current_workflow
            st.write(f"Name: {workflow.name}")
            workflow.description = st.text_area("Description:", value=workflow.description or "", key="tab2_workflow_description", on_change=handle_workflow_description_change)
            workflow.type = st.text_input("Type:", value=workflow.type, key="tab2_workflow_type", on_change=handle_workflow_type_change)
            workflow.summary_method = st.text_input("Summary Method:", value=workflow.summary_method, key="tab2_workflow_summary_method", on_change=handle_workflow_summary_method_change)

            # Add more workflow properties as needed

            # Display agents, sender, and receiver information
            st.write("Agents:")
            for agent in workflow.agents:
                st.write(f"- {agent.name}")

            with st.container(border=True):
                st.write("Sender:")
                st.write(f"Type: {workflow.sender.type}")
                st.write(f"User ID: {workflow.sender.user_id}")

            with st.container(border=True):
                st.write("Receiver:")
                st.write(f"Type: {workflow.receiver.type}")
                st.write(f"User ID: {workflow.receiver.user_id}")

    with agentTab:
        display_agent_dropdown()

        if st.session_state.current_agent is not None:
            # Display the properties of the current agent
            agent = st.session_state.current_agent
            st.write("<div style='color:#33FFFC; font-weight:bold; text-align:right; width:100%;'>AGENT PROPERTIES</div>", unsafe_allow_html=True)
            st.write(f"<div style='text-align:right; width:100%;'>Timestamp: {agent.timestamp}</div>", unsafe_allow_html=True)

            agent.description = st.text_area("Description:", value=agent.description or "", key=f"agent_description_{agent.name}", on_change=handle_agent_property_change)
            agent.role = st.text_input("Role:", value=agent.role or "", key=f"agent_role_{agent.name}", on_change=handle_agent_property_change)
            agent.goal = st.text_input("Goal:", value=agent.goal or "", key=f"agent_goal_{agent.name}", on_change=handle_agent_property_change)
            agent.backstory = st.text_area("Backstory:", value=agent.backstory or "", key=f"agent_backstory_{agent.name}", on_change=handle_agent_property_change)

    with toolTab:
        display_tool_dropdown()

        if st.session_state.current_tool is not None:
            # Display the properties of the current tool
            tool = st.session_state.current_tool
            st.write("<div style='color:#33FFFC; font-weight:bold; text-align:right; width:100%;'>TOOL PROPERTIES</div>", unsafe_allow_html=True)
            st.write(f"<div style='text-align:right; width:100%;'>Timestamp: {tool.timestamp}</div>", unsafe_allow_html=True)
            
            tool.content = st.text_area("Content:", value=tool.content, key=f"tool_content_{tool.name}", on_change=handle_tool_property_change)
            tool.title = st.text_input("Title:", value=tool.title, key=f"tool_title_{tool.name}", on_change=handle_tool_property_change)
            tool.description = st.text_input("Description:", value=tool.description or "", key=f"tool_description_{tool.name}", on_change=handle_tool_property_change)
            tool.file_name = st.text_input("File Name:", value=tool.file_name, key=f"tool_file_name_{tool.name}", on_change=handle_tool_property_change)
            tool.user_id = st.text_input("User ID:", value=tool.user_id, key=f"tool_user_id_{tool.name}", on_change=handle_tool_property_change)

    with settingsTab:
        st.write("Settings")
        
        provider_classes = load_provider_classes()
        default_provider = st.session_state.default_provider if st.session_state.default_provider else ""
        default_provider_index = provider_classes.index(default_provider) if default_provider in provider_classes else 0
        selected_provider = st.selectbox("Default Provider", provider_classes, index=default_provider_index, key="default_provider", on_change=handle_default_provider_change)

        if selected_provider:
            tmp = "_Provider"
            api_key = st.text_input("Default Provider's API Key:", type="password")
            st.session_state.default_provider_key = api_key

            if st.session_state.default_provider_key or os.environ.get(f"{selected_provider.replace(tmp, '').upper()}_API_KEY"):
                if os.environ.get(f"{selected_provider.replace(tmp, '').upper()}_API_KEY"):
                    st.session_state.default_provider_key = os.environ.get(f"{selected_provider.replace(tmp, '').upper()}_API_KEY")

                provider_module = importlib.import_module(f"providers.{selected_provider.lower()}")
                provider_class = getattr(provider_module, selected_provider)
                provider = provider_class(api_url="", api_key=st.session_state.default_provider_key)
                
                try:
                    if not st.session_state.available_models or len(st.session_state.available_models) == 0:
                        available_models = provider.get_available_models()
                    available_models = sorted(available_models)
                    selected_model = st.selectbox("Select Default Model", available_models)
                    st.session_state.selected_model = selected_model
                except Exception as e:
                    st.error(f"Error retrieving available models: {str(e)}")

        selected_framework = st.selectbox("Default Framework", ["Autogen", "CrewAI", "Bob's Agent Maker", "AI-Mart", "Geeks'R'Us"], key="default_framework")

    with debugTab:
        # Display the debug tab content
        if DEBUG:
            st.write("Debug Information")
            
            # Iterate over all session state variables
            for key, value in st.session_state.items():
                
                # Check if the value is an instance of specific classes
                if isinstance(value, (ProjectBaseModel, WorkflowBaseModel, AgentBaseModel, ToolBaseModel)):
                    # Display the properties and values of the object
                    for prop, prop_value in value.__dict__.items():
                        st.write(f"{prop}: {prop_value}")
                else:
                    # Display the value directly
                    st.write(f"{key}:\n\r {value}")
                
                st.write("---")

def display_agent_dropdown():
    if DEBUG:
        print("display_agent_dropdown()")
    if st.session_state.current_agent is None:
        # Display the agents dropdown
        agent_names = AgentBaseModel.load_agents()
        selected_agent = st.selectbox(
            "Agents",
            ["Select..."] + ["Create manually..."] + ["Create with AI..."] + agent_names,
            key="agent_dropdown",
            on_change=handle_agent_selection,
        )

        if selected_agent == "Select...":
            return
        if selected_agent == "Create manually...":
            # Show the manual agent creation input field
            st.text_input("Agent Name:", key="agent_name_input", on_change=handle_agent_selection)
        elif selected_agent == "Create with AI...":
            # Show the AI-assisted agent creation input field
            st.text_input("What should this new agent do?", key="agent_creation_input", on_change=handle_ai_agent_creation)
    else:
        st.session_state.current_agent.name = st.text_input(
            "Agent Name:",
            value=st.session_state.current_agent.name,
            key="agent_name_edit",
            on_change=handle_agent_property_change,
        )
        if st.button("CLOSE THIS AGENT"):
            handle_agent_close()


def display_project_dropdown():
    if DEBUG:
        print("display_project_dropdown()")
    if st.session_state.current_project is None:
        # Display the projects dropdown
        project_names = ProjectBaseModel.load_projects()
        selected_project = st.selectbox(
            "Projects",
            ["Select..."] + ["Create..."] + project_names,
            key="project_dropdown",
            on_change=handle_project_selection,
        )

        if selected_project == "Select...":
            return
        if selected_project == "Create...":
            # Show the create project input field
            st.text_input("Project Name:", key="project_name_input", on_change=handle_project_selection)
    else:
        # Display the selected project name as an editable text input
        st.session_state.current_project.name = st.text_input(
            "Project Name:",
            value=st.session_state.current_project.name,
            key="project_name_edit",
            on_change=handle_project_name_change,
        )
        if st.button("CLOSE THIS PROJECT"):
            handle_project_close()


def display_tool_dropdown():
    if DEBUG:
        print("display_tool_dropdown()")
    if st.session_state.current_tool is None:
        # Display the tools dropdown
        tool_names = ToolBaseModel.load_tools()
        selected_tool = st.selectbox(
            "Tools",
            ["Select..."] + ["Create manually..."] + ["Create with AI..."] + tool_names,
            key="tool_dropdown",
            on_change=handle_tool_selection,
        )

        if selected_tool == "Select...":
            return
        if selected_tool == "Create manually...":
            # Show the manual tool creation input field
            st.text_input("Tool Name:", key="tool_name_input", on_change=handle_tool_selection)
        elif selected_tool == "Create with AI...":
            # Show the AI-assisted tool creation input field
            st.text_input("What should this new tool do?", key="tool_creation_input", on_change=handle_ai_tool_creation)
    else:
        st.session_state.current_tool.name = st.text_input(
            "Tool Name:",
            value=st.session_state.current_tool.name,
            key="tool_name_edit",
            on_change=handle_tool_property_change,
        )
        if st.button("CLOSE THIS TOOL"):
            handle_tool_close()


def display_workflow_dropdown():
    if DEBUG:
        print("display_workflow_dropdown()")
    if st.session_state.current_workflow is None:
        # Display the workflows dropdown
        workflow_names = WorkflowBaseModel.load_workflows()
        selected_workflow = st.selectbox(
            "Workflows",
            ["Select..."] + ["Create..."] + workflow_names,
            key="workflow_dropdown",
            on_change=handle_workflow_selection,
        )
        if selected_workflow == "Select...":
            return
        if selected_workflow == "Create...":
            # Show the create workflow input field
            st.text_input("Workflow Name:", key="workflow_name_input", on_change=handle_workflow_selection)
    else:
        st.session_state.current_workflow.name = st.text_input(
            "Workflow Name:",
            value=st.session_state.current_workflow.name,
            key="workflow_name_edit",
            on_change=handle_workflow_name_change,
        )
        if st.button("CLOSE THIS WORKFLOW"):
            handle_workflow_close()


def sidebar_begin():
    if DEBUG:
        print("sidebar_begin()")

    st.sidebar.write("<div class='title'>AutoGrok™ <br/> Universal AI Agents Made Easy. <br/> Eventually.</div><p/>", unsafe_allow_html=True)
    st.sidebar.write("(We're putting the 'mental' in 'experimental'.)")
    st.sidebar.write("No need to report what's broken, we know.")
    # display_project_dropdown()  
    # display_workflow_dropdown()
    
```

# tools\py\document_indexer.py

```python
#  Thanks to MADTANK:  https://github.com/madtank
#  README:  https://github.com/madtank/autogenstudio-skills/blob/main/rag/README.md

import argparse
import bs4
import csv
import json
import os
import pickle
import re
import traceback
from typing import Dict, List, Literal, Tuple

try:
    import tiktoken
    from langchain_community.embeddings import HuggingFaceEmbeddings
    from langchain_community.vectorstores import FAISS
except ImportError:
    raise ImportError("Please install the dependencies first.")


def chunk_str_overlap(
    s: str,
    separator: chr = "\n",
    num_tokens: int = 64,
    step_tokens: int = 64,
    encoding: tiktoken.Encoding = None,
) -> List[str]:
    """
    Split a string into chunks with overlap
    :param s: the input string
    :param separator: the separator to split the string
    :param num_tokens: the number of tokens in each chunk
    :param step_tokens: the number of tokens to step forward
    :param encoding: the encoding to encode the string
    """
    assert step_tokens <= num_tokens, (
        f"The number of tokens {num_tokens} in each chunk " f"should be larger than the step size {step_tokens}."
    )

    lines = s.split(separator)
    chunks = dict()
    final_chunks = []

    if len(lines) == 0:
        return []

    first_line = lines[0]
    first_line_size = len(encoding.encode(first_line))

    chunks[0] = [first_line, first_line_size]

    this_step_size = first_line_size

    for i in range(1, len(lines)):
        line = lines[i]
        line_size = len(encoding.encode(line))

        to_pop = []
        for key in chunks:
            if chunks[key][1] + line_size > num_tokens:
                to_pop.append(key)
            else:
                chunks[key][0] += f"{separator}{line}"
                chunks[key][1] += line_size
        final_chunks += [chunks.pop(key)[0] for key in to_pop]

        if this_step_size + line_size > step_tokens:
            chunks[i] = [line, line_size]
            this_step_size = 0
        this_step_size += line_size

    max_remained_chunk = ""
    max_remained_chunk_size = 0
    for key in chunks:
        if chunks[key][1] > max_remained_chunk_size:
            max_remained_chunk_size = chunks[key][1]
            max_remained_chunk = chunks[key][0]
    if max_remained_chunk_size > 0:
        final_chunks.append(max_remained_chunk)

    return final_chunks


def get_title(
    file_name: str,
    prop="title: ",
) -> str:
    """
    Get the title of a file
    :param file_name: the file name
    :param prop: the property to get the title
    """
    with open(file_name, encoding="utf-8", errors="ignore") as f_in:
        for line in f_in:
            line = line.strip()
            if line and (line.startswith(prop) or any([c.isalnum() for c in line])):
                return line
    return ""


def extract_text_from_file(
    file: str,
    file_type: Literal["pdf", "docx", "csv", "pptx"],
) -> Tuple[str, str]:
    """
    Extract text from a file in pdf, docx, csv or pptx format
    :param file: the file path
    :param file_type: the extension of the file
    """
    if file_type == "pdf":
        try:
            from pypdf import PdfReader
        except ImportError:
            raise ImportError("Please install pypdf first.")
        # Extract text from pdf using PyPDF2
        reader = PdfReader(file)
        extracted_text = " ".join([page.extract_text() for page in reader.pages])
        title = extracted_text.split("\n")[0]
    elif file_type == "docx":
        try:
            import docx2txt
        except ImportError:
            raise ImportError("Please install docx2txt first.")
        # Extract text from docx using docx2txt
        extracted_text = docx2txt.process(file)
        title = extracted_text.split("\n")[0]
    elif file_type == "csv":
        # Extract text from csv using csv module
        extracted_text = ""
        title = ""
        reader = csv.reader(file)
        for row in reader:
            extracted_text += " ".join(row) + "\n"
    elif file_type == "pptx":
        try:
            import pptx
        except ImportError:
            raise ImportError("Please install python-pptx first.")
        extracted_text = ""
        no_title = True
        title = ""
        presentation = pptx.Presentation(file)
        for slide in presentation.slides:
            for shape in slide.shapes:
                if shape.has_text_frame:
                    for paragraph in shape.text_frame.paragraphs:
                        for run in paragraph.runs:
                            extracted_text += run.text + " "
                            if no_title and len(run.text) > 10:
                                title = run.text
                                no_title = False
                    extracted_text += "\n"
    else:
        # Unsupported file type
        raise ValueError(f"Unsupported file type: {file_type}")

    return title[:100], extracted_text


def text_parser(
    read_file: str,
) -> Tuple[str, str]:
    """
    Returns the title, parsed text and a BeautifulSoup object with different file extension
    : param read_file: the input file with a given extension
    : return: the title, parsed text and a BeautifulSoup object, the BeautifulSoup object is used to get the document
        link from the html files
    """
    filename, extension = os.path.splitext(read_file)
    extension = extension.lstrip(".")
    title = filename
    soup = None
    supported_extensions = ["md", "markdown", "html", "htm", "txt", "json", "jsonl"]
    other_extensions = ["docx", "pptx", "pdf", "csv"]

    # utf-8-sig will treat BOM header as a metadata of a file not a part of the file content
    default_encoding = "utf-8-sig"

    if extension in ("md", "markdown", "txt"):
        title = get_title(read_file)
        with open(read_file, "r", encoding=default_encoding, errors="ignore") as f:
            text = f.read()
    elif extension in ("html", "htm"):
        from bs4 import BeautifulSoup

        with open(read_file, "r", encoding=default_encoding, errors="ignore") as f:
            soup = BeautifulSoup(f, "html.parser")
        title = next(soup.stripped_strings)[:100]
        text = soup.get_text("\n")
    # read json/jsonl file in and convert each json to a row of string
    elif extension in ("json", "jsonl"):
        try:
            with open(read_file, "r", encoding=default_encoding, errors="ignore") as f:
                data = json.load(f) if extension == "json" else [json.loads(line) for line in f]
        except:
            # json file encoding issue, skip this file
            return title, ""

        if isinstance(data, dict):
            text = json.dumps(data)
        elif isinstance(data, list):
            content_list = [json.dumps(each_json) for each_json in data]
            text = "\n".join(content_list)
            title = filename
    elif extension in other_extensions:
        title, text = extract_text_from_file(read_file, extension)
    else:  # no support for other format
        print(
            f"Not support for file with extension: {extension}. "
            f"The supported extensions are {supported_extensions}",
        )
        return title, ""

    output_text = re.sub(r"\n{3,}", "\n\n", text)
    # keep whitespaces for formatting
    output_text = re.sub(r"-{3,}", "---", output_text)
    output_text = re.sub(r"\*{3,}", "***", output_text)
    output_text = re.sub(r"_{3,}", "___", output_text)

    return title, output_text


def chunk_document(
    doc_path: str,
    chunk_size: int,
    chunk_step: int,
) -> Tuple[int, List[str], List[Dict[str, str]], Dict[str, int]]:
    """
    Split documents into chunks
    :param doc_path: the path of the documents
    :param chunk_size: the size of the chunk
    :param chunk_step: the step size of the chunk
    """
    texts = []
    metadata_list = []
    file_count = 0
    chunk_id_to_index = dict()

    enc = tiktoken.encoding_for_model("gpt-3.5-turbo")

    # traverse all files under dir
    print("Split documents into chunks...")
    for root, dirs, files in os.walk(doc_path):
        for name in files:
            f = os.path.join(root, name)
            print(f"Reading {f}")
            try:
                title, content = text_parser(f)
                file_count += 1
                if file_count % 100 == 0:
                    print(f"{file_count} files read.")

                if len(content) == 0:
                    continue

                chunks = chunk_str_overlap(
                    content.strip(),
                    num_tokens=chunk_size,
                    step_tokens=chunk_step,
                    separator="\n",
                    encoding=enc,
                )
                source = os.path.sep.join(f.split(os.path.sep)[4:])
                for i in range(len(chunks)):
                    # custom metadata if needed
                    metadata = {
                        "source": source,
                        "title": title,
                        "chunk_id": i,
                    }
                    chunk_id_to_index[f"{source}_{i}"] = len(texts) + i
                    metadata_list.append(metadata)
                texts.extend(chunks)
            except Exception as e:
                print(f"Error encountered when reading {f}: {traceback.format_exc()} {e}")
    return file_count, texts, metadata_list, chunk_id_to_index


if __name__ == "__main__":
    # parse arguments
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "-d",
        "--doc_path",
        help="the path of the documents",
        type=str,
        default="documents",
    )
    parser.add_argument(
        "-c",
        "--chunk_size",
        help="the size of the chunk",
        type=int,
        default=64,
    )
    parser.add_argument(
        "-s",
        "--chunk_step",
        help="the step size of the chunk",
        type=int,
        default=64,
    )
    parser.add_argument(
        "-o",
        "--output_path",
        help="the path of the output",
        type=str,
        default="knowledge",
    )
    args = parser.parse_args()

    file_count, texts, metadata_list, chunk_id_to_index = chunk_document(
        doc_path=args.doc_path,
        chunk_size=args.chunk_size,
        chunk_step=args.chunk_step,
    )
    embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
    vectorstore = FAISS.from_texts(
        texts=texts,
        metadatas=metadata_list,
        embedding=embeddings,
    )
    vectorstore.save_local(folder_path=args.output_path)
    with open(os.path.join(args.output_path, "chunk_id_to_index.pkl"), "wb") as f:
        pickle.dump(chunk_id_to_index, f)
    print(f"Saved vectorstore to {args.output_path}")
```

# tools\py\document_retriever.py

```python
# #  Thanks to MADTANK:  https://github.com/madtank
# #  README:  https://github.com/madtank/autogenstudio-skills/blob/main/rag/README.md

# import os
# import pickle
# import json
# import argparse

# try:
#     import tiktoken
#     from langchain_community.embeddings import HuggingFaceEmbeddings
#     from langchain_community.vectorstores import FAISS
# except ImportError:
#     raise ImportError("Please install langchain-community first.")

# # Configuration - Users/AI skill developers must update this path to their specific index folder
# # To test with sample data set index_folder to "knowledge"
# CONFIG = {
#     "index_folder": "rag/knowledge",  # TODO: Update this path before using
# }

# class DocumentRetriever:
#     def __init__(self, index_folder):
#         self.index_folder = index_folder
#         self.vectorstore = None
#         self.chunk_id_to_index = None
#         self.embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
#         self._init()
#         self.enc = tiktoken.encoding_for_model("gpt-3.5-turbo")

#     def _init(self):
#         self.vectorstore = FAISS.load_local(
#             folder_path=self.index_folder,
#             embeddings=self.embeddings,
#         )
#         with open(os.path.join(self.index_folder, "chunk_id_to_index.pkl"), "rb") as f:
#             self.chunk_id_to_index = pickle.load(f)

#     def __call__(self, query: str, size: int = 5, target_length: int = 256):
#         if self.vectorstore is None:
#             raise Exception("Vectorstore not initialized")

#         result = self.vectorstore.similarity_search(query=query, k=size)
#         expanded_chunks = self.do_expand(result, target_length)

#         return json.dumps(expanded_chunks, indent=4)

#     def do_expand(self, result, target_length):
#         expanded_chunks = []
#         # do expansion
#         for r in result:
#             source = r.metadata["source"]
#             chunk_id = r.metadata["chunk_id"]
#             content = r.page_content

#             expanded_result = content
#             left_chunk_id, right_chunk_id = chunk_id - 1, chunk_id + 1
#             left_valid, right_valid = True, True
#             chunk_ids = [chunk_id]
#             while True:
#                 current_length = len(self.enc.encode(expanded_result))
#                 if f"{source}_{left_chunk_id}" in self.chunk_id_to_index:
#                     chunk_ids.append(left_chunk_id)
#                     left_chunk_index = self.vectorstore.index_to_docstore_id[
#                         self.chunk_id_to_index[f"{source}_{left_chunk_id}"]
#                     ]
#                     left_chunk = self.vectorstore.docstore.search(left_chunk_index)
#                     encoded_left_chunk = self.enc.encode(left_chunk.page_content)
#                     if len(encoded_left_chunk) + current_length < target_length:
#                         expanded_result = left_chunk.page_content + expanded_result
#                         left_chunk_id -= 1
#                         current_length += len(encoded_left_chunk)
#                     else:
#                         expanded_result += self.enc.decode(
#                             encoded_left_chunk[-(target_length - current_length) :],
#                         )
#                         current_length = target_length
#                         break
#                 else:
#                     left_valid = False

#                 if f"{source}_{right_chunk_id}" in self.chunk_id_to_index:
#                     chunk_ids.append(right_chunk_id)
#                     right_chunk_index = self.vectorstore.index_to_docstore_id[
#                         self.chunk_id_to_index[f"{source}_{right_chunk_id}"]
#                     ]
#                     right_chunk = self.vectorstore.docstore.search(right_chunk_index)
#                     encoded_right_chunk = self.enc.encode(right_chunk.page_content)
#                     if len(encoded_right_chunk) + current_length < target_length:
#                         expanded_result += right_chunk.page_content
#                         right_chunk_id += 1
#                         current_length += len(encoded_right_chunk)
#                     else:
#                         expanded_result += self.enc.decode(
#                             encoded_right_chunk[: target_length - current_length],
#                         )
#                         current_length = target_length
#                         break
#                 else:
#                     right_valid = False

#                 if not left_valid and not right_valid:
#                     break

#             expanded_chunks.append(
#                 {
#                     "chunk": expanded_result,
#                     "metadata": r.metadata,
#                     # "length": current_length,
#                     # "chunk_ids": chunk_ids
#                 },
#             )
#         return expanded_chunks

# # Example Usage
# if __name__ == "__main__":
#     parser = argparse.ArgumentParser(description='Retrieve documents based on a query.')
#     parser.add_argument('query', nargs='?', type=str, help='The query to retrieve documents for.')
#     args = parser.parse_args()

#     if not args.query:
#         parser.print_help()
#         print("Error: No query provided.")
#         exit(1)

#     # Ensure the index_folder path is correctly set in CONFIG before proceeding
#     index_folder = CONFIG["index_folder"]
#     if index_folder == "path/to/your/knowledge/directory":
#         print("Error: Index folder in CONFIG has not been set. Please update it to your index folder path.")
#         exit(1)

#     # Instantiate and use the DocumentRetriever with the configured index folder
#     retriever = DocumentRetriever(index_folder=index_folder)
#     query = args.query
#     size = 5  # Number of results to retrieve
#     target_length = 256  # Target length of expanded content
#     results = retriever(query, size, target_length)
#     print(results)
```

# tools\py\fetch_web_content.py

```python
#  Thanks to MADTANK:  https://github.com/madtank

from typing import Optional
import requests
import collections
collections.Callable = collections.abc.Callable
from bs4 import BeautifulSoup

def fetch_web_content(url: str) -> Optional[str]:
    """
    Fetches the text content from a website.

    Args:
        url (str): The URL of the website.

    Returns:
        Optional[str]: The content of the website.
    """
    try:
        # Send a GET request to the URL
        response = requests.get(url)

        # Check for successful access to the webpage
        if response.status_code == 200:
            # Parse the HTML content of the page using BeautifulSoup
            soup = BeautifulSoup(response.text, "html.parser")

            # Extract the content of the <body> tag
            body_content = soup.body

            if body_content:
                # Return all the text in the body tag, stripping leading/trailing whitespaces
                return " ".join(body_content.get_text(strip=True).split())
            else:
                # Return None if the <body> tag is not found
                return None
        else:
            # Return None if the status code isn't 200 (success)
            return None
    except requests.RequestException:
        # Return None if any request-related exception is caught
        return None
```

# tools\py\generate_sd_images.py

```python
# Thanks to marc-shade:  https://github.com/marc-shade
# Ollama only?  -jjg

from typing import List
import json
import requests
import io
import base64
from PIL import Image
from pathlib import Path
import uuid # Import the uuid library

# Format: protocol://server:port
base_url = "http://0.0.0.0:7860"

def generate_sd_images(query: str, image_size: str = "512x512", team_name: str = "default") -> List[str]:
    """
    Function to paint, draw or illustrate images based on the users query or request. 
    Generates images locally with the automatic1111 API and saves them to disk.  
    Use the code below anytime there is a request to create an image.

    :param query: A natural language description of the image to be generated.
    :param image_size: The size of the image to be generated. (default is "512x512")
    :param team_name: The name of the team to associate the image with.
    :return: A list containing a single filename for the saved image.
    """
    # Split the image size string at "x"
    parts = image_size.split("x")
    image_width = parts[0]
    image_height = parts[1]

    # list of file paths returned to AutoGen
    saved_files = []

    payload = {
        "prompt": query,
        "steps": 40,
        "cfg_scale": 7,
        "denoising_strength": 0.5,
        "sampler_name": "DPM++ 2M Karras",
        "n_iter": 1,
        "batch_size": 1, # Ensure only one image is generated per batch
        "override_settings": {
             'sd_model_checkpoint': "starlightAnimated_v3",
        }
    }

    api_url = f"{base_url}/sdapi/v1/txt2img"
    response = requests.post(url=api_url, json=payload)

    if response.status_code == 200:
        r = response.json()
        # Access only the final generated image (index 0)
        encoded_image = r['images'][0] 

        image = Image.open(io.BytesIO(base64.b64decode(encoded_image.split(",", 1)[0])))
        
        # --- Generate a unique filename with team name and UUID ---
        unique_id = str(uuid.uuid4())[:8] # Get a short UUID
        file_name = f"images/{team_name}_{unique_id}_output.png"
        
        file_path = Path(file_name)
        image.save(file_path)
        print(f"Image saved to {file_path}")

        saved_files.append(str(file_path))
    else:
        print(f"Failed to download the image from {api_url}")

    return saved_files
```

# tools\py\get_complementary_colors.py

```python
# Tool filename: complementary_colors.py
# Import necessary module(s)
import colorsys

def get_complementary_colors(color):
    # docstrings
    """
    Returns a list of complementary colors for the given color.

    Parameters:
    color (str): The color in hexadecimal format (e.g., '#FF0000' for red).

    Returns:
    list: A list of complementary colors in hexadecimal format.
    """

    # Body of tool
    # Convert the color from hexadecimal to RGB
    r, g, b = tuple(int(color.lstrip('#')[i:i+2], 16) for i in (0, 2, 4))
    # Convert RGB to HSV
    h, s, v = colorsys.rgb_to_hsv(r/255, g/255, b/255)
    # Calculate the complementary hue
    h_compl = (h + 0.5) % 1
    # Convert the complementary hue back to RGB
    r_compl, g_compl, b_compl = colorsys.hsv_to_rgb(h_compl, 1, 1)
    # Convert RGB to hexadecimal
    color_compl = '#{:02x}{:02x}{:02x}'.format(int(r_compl*255), int(g_compl*255), int(b_compl*255))
    # Return the complementary color
    return [color_compl]

    # Example usage:
    # color = '#FF0000'
    # print(get_complementary_colors(color))
```

# tools\py\get_weather.py

```python
import requests
from typing import Optional

def get_weather(zipcode: str, api_key: str) -> Optional[dict]:
    """
    Fetches the current weather for the given ZIP code using the OpenWeatherMap API.

    Args:
        zipcode (str): The ZIP code for which to fetch the weather.
        api_key (str): Your OpenWeatherMap API key.

    Returns:
        Optional[dict]: A dictionary containing the weather information, or None if an error occurs.
    """
    base_url = "http://api.openweathermap.org/data/2.5/weather"
    params = {
        "zip": zipcode,
        "appid": api_key,
        "units": "imperial"  # Use "metric" for Celsius
    }
    
    try:
        response = requests.get(base_url, params=params)
        response.raise_for_status()  # Raise an HTTPError for bad responses
        return response.json()
    except requests.RequestException as e:
        print(f"An error occurred: {e}")
        return None

# Example usage:
# api_key = "your_openweathermap_api_key"
# weather = get_weather("94040", api_key)
# print(weather)

```

# tools\py\plot_diagram.py

```python
#  Thanks to MADTANK:  https://github.com/madtank

import os
import matplotlib.pyplot as plt
import matplotlib.patches as patches

# Function to draw the geometric structure with customizable file name
def draw_geometric_structure(file_name, base_circles=4, base_circle_color='blue', top_circle_color='orange', line_color='grey', line_width=2):

    # Define the directory and save path using the file_name parameter
    directory = 'diagrams'
    if not os.path.exists(directory):
        os.makedirs(directory)
    save_path = f'{directory}/{file_name}.png'
    
    fig, ax = plt.subplots()

    # Draw base circles
    for i in range(base_circles):
        circle = patches.Circle((i * 1.5, 0), 0.5, color=base_circle_color)
        ax.add_patch(circle)

    # Draw top circle
    top_circle = patches.Circle(((base_circles - 1) * 0.75, 2), 0.6, color=top_circle_color)
    ax.add_patch(top_circle)

    # Draw lines
    for i in range(base_circles):
        line = plt.Line2D([(i * 1.5), ((base_circles - 1) * 0.75)], [0, 2], color=line_color, linewidth=line_width)
        ax.add_line(line)

    # Set limits and aspect
    ax.set_xlim(-1, base_circles * 1.5)
    ax.set_ylim(-1, 3)
    ax.set_aspect('equal')

    # Remove axes
    ax.axis('off')

    # Save the plot to the specified path
    plt.savefig(save_path, bbox_inches='tight', pad_inches=0)
    plt.close()

    # Return the path for verification
    return save_path

# Example usage:
#file_name = 'custom_geometric_structure'
#image_path = draw_geometric_structure(file_name, base_circles=8, base_circle_color='blue', top_circle_color='orange', line_color='grey', line_width=2)
```

# tools\py\save_file_to_disk.py

```python
# Thanks to aj47:  https://github.com/aj47

import os

def save_file_to_disk(contents, file_name):
    """
    Saves the given contents to a file with the given file name.

    Parameters:
    contents (str): The string contents to save to the file.
    file_name (str): The name of the file, including its extension.

    Returns:
    str: A message indicating the success of the operation.
    """
    # Ensure the directory exists; create it if it doesn't
    directory = os.path.dirname(file_name)
    if directory and not os.path.exists(directory):
        os.makedirs(directory)

    # Write the contents to the file
    with open(file_name, 'w') as file:
        file.write(contents)
    
    return f"File '{file_name}' has been saved successfully."

# Example usage:
# contents_to_save = "Hello, world!"
# file_name = "example.txt"
# print(save_file_to_disk(contents_to_save, file_name))
```

# tools\py\slackoverflow_teams.py

```python
# #  Thanks to MADTANK:  https://github.com/madtank
# #  README:  https://github.com/madtank/autogenstudio-skills/blob/main/stackoverflow_teams/README.md

# import os
# import requests
# import json
# import sys

# class StackOverflowTeamsSearcher:
#     def __init__(self):
#         self.api_key = os.getenv("STACK_OVERFLOW_TEAMS_API_KEY")
#         if not self.api_key:
#             raise ValueError("API key not found in environment variables")
#         self.base_url = "https://api.stackoverflowteams.com/2.3/search"
#         self.headers = {"X-API-Access-Token": self.api_key}

#     def search(self, query, team_name):
#         params = {"intitle": query, "team": team_name}
#         response = requests.get(self.base_url, headers=self.headers, params=params)

#         if response.status_code != 200:
#             print(f"Error: Received status code {response.status_code}")
#             print(response.text)
#             return None

#         try:
#             data = response.json()
#             simplified_output = []
#             for item in data['items']:
#                 question = {"question": item['title']}
#                 if 'accepted_answer_id' in item:
#                     answer_id = item['accepted_answer_id']
#                     answer_url = f"https://api.stackoverflowteams.com/2.3/answers/{answer_id}"
#                     answer_params = {"team": team_name, "filter": "withbody"}
#                     answer_response = requests.get(answer_url, headers=self.headers, params=answer_params)
#                     if answer_response.status_code == 200:
#                         answer_data = answer_response.json()
#                         first_item = answer_data['items'][0]
#                         if 'body' in first_item:
#                             answer_text = first_item['body']
#                             question['answer'] = answer_text
# #                        else:
# #                            print(f"Question {item['link']} has no answer body")
# #                    else:
# #                        print(f"Error: Received status code {answer_response.status_code}")
# #                        print(answer_response.text)
# #                else:
# #                    print(f"Question {item['link']} has no answer")
#                 simplified_output.append(question)
#             return json.dumps(simplified_output, indent=4)  # Pretty-printing
#         except ValueError as e:
#             print(f"Error parsing JSON: {e}")
#             print("Response text:", response.text)
#             return None

# # Example Usage
# if __name__ == "__main__":
#     if len(sys.argv) < 2:
#         print("Usage: python stackoverflow_teams.py <query>")
#         sys.exit(1)

#     query = sys.argv[1]
#     team_name = "yourteamname"  # TODO Set your team name here
#     # Instantiate and use the StackOverflowTeamsSearcher with the query string passed in
#     searcher = StackOverflowTeamsSearcher()
#     results = searcher.search(query, team_name)
#     print(results)
```

# tools\py\slack_search.py

```python
# #  Thanks to MADTANK:  https://github.com/madtank
# #  README:  https://github.com/madtank/autogenstudio-skills/blob/main/slack/README.md

# import os
# import requests
# import json
# import re
# import sys

# class SlackSearcher:
#     def __init__(self):
#         self.api_token = os.getenv("SLACK_API_TOKEN")
#         if not self.api_token:
#             raise ValueError("Slack API token not found in environment variables")
#         self.base_url = "https://slack.com/api"
#         self.headers = {"Authorization": f"Bearer {self.api_token}"}
#         # Replace these example channel names with the actual channel names you want to search
#         self.channel_names = ["general", "random"]

#     def search(self, query):
#         query_with_channels = self.build_query_with_channels(query)
#         search_url = f"{self.base_url}/search.messages"
#         params = {"query": query_with_channels}
#         response = requests.get(search_url, headers=self.headers, params=params)

#         if response.status_code != 200:
#             print(f"Error: Received status code {response.status_code}")
#             print(response.text)
#             return None

#         try:
#             data = response.json()
#             if not data['ok']:
#                 print(f"Error: {data['error']}")
#                 return None

#             simplified_output = []
#             for message in data['messages']['matches']:
#                 simplified_message = {
#                     "user": message['user'],
#                     "text": message['text'],
#                     "permalink": message['permalink']
#                 }
#                 thread_ts = self.extract_thread_ts(message['permalink'])
#                 if thread_ts:
#                     thread_messages = self.get_thread_messages(message['channel']['id'], thread_ts)
#                     simplified_message['thread'] = thread_messages
#                 simplified_output.append(simplified_message)
#             return json.dumps(simplified_output, indent=4)  # Pretty-printing
#         except ValueError as e:
#             print(f"Error parsing JSON: {e}")
#             print("Response text:", response.text)
#             return None

#     def build_query_with_channels(self, query):
#         channel_queries = [f"in:{channel}" for channel in self.channel_names]
#         return f"{query} {' '.join(channel_queries)}"

#     def extract_thread_ts(self, permalink):
#         match = re.search(r"thread_ts=([0-9.]+)", permalink)
#         return match.group(1) if match else None

#     def get_thread_messages(self, channel_id, thread_ts):
#         thread_url = f"{self.base_url}/conversations.replies"
#         params = {"channel": channel_id, "ts": thread_ts}
#         response = requests.get(thread_url, headers=self.headers, params=params)

#         if response.status_code != 200 or not response.json()['ok']:
#             print(f"Error fetching thread messages: {response.text}")
#             return []

#         thread_messages = []
#         for message in response.json()['messages']:
#             if message['ts'] != thread_ts:  # Exclude the parent message
#                 thread_messages.append({
#                     "user": message['user'],
#                     "text": message['text']
#                 })
#         return thread_messages

# # Example Usage
# if __name__ == "__main__":
#     if len(sys.argv) < 2:
#         print("Usage: python slack_search.py <query>")
#         sys.exit(1)

#     query = sys.argv[1]
#     searcher = SlackSearcher()
#     results = searcher.search(query)
#     print(results)
```

# tools\py\test.py

```python
# bfrglz; = 
# return

# import json
```

# tools\py\webscrape.py

```python
#  Thanks to MADTANK:  https://github.com/madtank

import requests
from bs4 import BeautifulSoup


def save_webpage_as_text(url, output_filename):
    # Send a GET request to the URL
    response = requests.get(url)
    
    # Initialize BeautifulSoup to parse the content
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Extract text from the BeautifulSoup object
    # You can adjust the elements you extract based on your needs
    text = soup.get_text(separator='\n', strip=True)
    
    # Save the extracted text to a file
    with open(output_filename, 'w', encoding='utf-8') as file:
        file.write(text)
    
    # Return the file path
    return output_filename


# Example usage:
# url = 'https://j.gravelle.us        /'
# output_filename = 'webpage_content.txt'
# file_path = save_webpage_as_text(url, output_filename)
# print("File saved at:", file_path)


# For a list of urls:
# urls = ['http://example.com', 'http://example.org']
# for i, url in enumerate(urls):
#     output_filename = f'webpage_content_{i}.txt'
#     save_webpage_as_text(url, output_filename)
```

# tools\py\web_search.py

```python
# #  Thanks to MADTANK:  https://github.com/madtank
# #  README:  https://github.com/madtank/autogenstudio-skills/blob/main/web_search/README.MD

# import requests
# from typing import List, Tuple, Optional

# # Define the structure of a search result entry
# ResponseEntry = Tuple[str, str, str]

# # Configuration variables for the web search function
# CONFIG = {
#     "api_provider": "google",  # or "bing"
#     "result_count": 3,
#     # For Google Search enter these values 
#     # Refer to readme for help:  https://github.com/madtank/autogenstudio-skills/blob/main/web_search/README.MD 
#     "google_api_key": "your_google_api_key_here",
#     "google_search_engine_id": "your_google_search_engine_id_here",
#     # Or Bing Search enter these values
#     "bing_api_key": "your_bing_api_key_here"
# }

# class WebSearch:
#     """
#     A class that encapsulates the functionality to perform web searches using
#     Google Custom Search API or Bing Search API based on the provided configuration.
#     """

#     def __init__(self, config: dict):
#         """
#         Initializes the WebSearch class with the provided configuration.

#         Parameters:
#         - config (dict): A dictionary containing configuration settings.
#         """
#         self.config = config

#     def search_query(self, query: str) -> Optional[List[ResponseEntry]]:
#         """
#         Performs a web search based on the query and configuration.

#         Parameters:
#         - query (str): The search query string.

#         Returns:
#         - A list of ResponseEntry tuples containing the title, URL, and snippet of each result.
#         """
#         api_provider = self.config.get("api_provider", "google")
#         result_count = int(self.config.get("result_count", 3))
#         try:
#             if api_provider == "google":
#                 return self._search_google(query, cnt=result_count)
#             elif api_provider == "bing":
#                 return self._search_bing(query, cnt=result_count)
#         except ValueError as e:
#             print(f"An error occurred: {e}")
#         except Exception as e:
#             print(f"An unexpected error occurred: {e}")
#         return None

#     def _search_google(self, query: str, cnt: int) -> Optional[List[ResponseEntry]]:
#         """
#         Performs a Google search and processes the results.
#         Parameters:
#         - query (str): The search query string.
#         - cnt (int): The number of search results to return.

#         Returns:
#         - A list of ResponseEntry tuples containing the title, URL, and snippet of each Google search result.
#         """
#         api_key = self.config.get("google_api_key")
#         search_engine_id = self.config.get("google_search_engine_id")
#         url = f"https://www.googleapis.com/customsearch/v1?key={api_key}&cx={search_engine_id}&q={query}"
#         if cnt > 0:
#             url += f"&num={cnt}"
#         response = requests.get(url)
#         if response.status_code == 200:
#             result_list: List[ResponseEntry] = []
#             for item in response.json().get("items", []):
#                 result_list.append((item["title"], item["link"], item["snippet"]))
#             return result_list
#         else:
#             print(f"Error with Google Custom Search API: {response.status_code}")
#             return None

#     def _search_bing(self, query: str, cnt: int) -> Optional[List[ResponseEntry]]:
#         """
#         Performs a Bing search and processes the results.

#         Parameters:
#         - query (str): The search query string.
#         - cnt (int): The number of search results to return.

#         Returns:
#         - A list of ResponseEntry tuples containing the name, URL, and snippet of each Bing search result.
#         """
#         api_key = self.config.get("bing_api_key")
#         url = f"https://api.bing.microsoft.com/v7.0/search?q={query}"
#         if cnt > 0:
#             url += f"&count={cnt}"
#         headers = {"Ocp-Apim-Subscription-Key": api_key}
#         response = requests.get(url, headers=headers)
#         if response.status_code == 200:
#             result_list: List[ResponseEntry] = []
#             for item in response.json().get("webPages", {}).get("value", []):
#                 result_list.append((item["name"], item["url"], item["snippet"]))
#             return result_list
#         else:
#             print(f"Error with Bing Search API: {response.status_code}")
#             return None

# # Remember to replace the placeholders in CONFIG with your actual API keys.
# # Example usage
# # search = WebSearch(CONFIG)
# # results = search.search_query("Example Query")
# # if results is not None:
# #     for title, link, snippet in results:
# #         print(title, link, snippet)
```

# agents\primary_assistant.yaml

```yaml
type: assistant
config:
  name: primary_assistant
  llm_config:
    config_list:
      - user_id: default
        timestamp: ""
        model: ""
        base_url: null
        api_type: null
        api_version: null
        description: OpenAI model configuration
    temperature: 0.1
    cache_seed: null
    timeout: null
    max_tokens: null
    extra_body: null
  human_input_mode: NEVER
  max_consecutive_auto_reply: 30
  system_message: |
    You are a helpful AI assistant. Solve tasks using your coding and language skills. In the following cases, suggest python code (in a python coding block) or shell script (in a sh coding block) for the user to execute. 
    1. When you need to collect info, use the code to output the info you need, for example, browse or search the web, download/read a file, print the content of a webpage or a file, get the current date/time, check the operating system. After sufficient info is printed and the task is ready to be solved based on your language skill, you can solve the task by yourself. 
    2. When you need to perform some task with code, use the code to perform the task and output the result. Finish the task smartly. Solve the task step by step if you need to. If a plan is not provided, explain your plan first. Be clear which step uses code, and which step uses your language skill. When using code, you must indicate the script type in the code block. The user cannot provide any other feedback or perform any other action beyond executing the code you suggest. The user can't modify your code. So do not suggest incomplete code which requires users to modify. Don't use a code block if it's not intended to be executed by the user. If you want the user to save the code in a file before executing it, put # filename: <filename> inside the code block as the first line. Don't include multiple code blocks in one response. Do not ask users to copy and paste the result. Instead, use 'print' function for the output when relevant. Check the execution result returned by the user. If the result indicates there is an error, fix the error and output the code again. Suggest the full code instead of partial code or code changes. If the error can't be fixed or if the task is not solved even after the code is executed successfully, analyze the problem, revisit your assumption, collect additional info you need, and think of a different approach to try. When you find an answer, verify the answer carefully. Include verifiable evidence in your response if possible. Reply 'TERMINATE' in the end when everything is done.
  is_termination_msg: null
  code_execution_config: null
  default_auto_reply: ""
  description: A primary assistant agent that writes plans and code to solve tasks. booger
timestamp: ""
user_id: default
skills:
  - title: fetch_web_content
    content: |
      from typing import Optional
      import requests
      import collections
      collections.Callable = collections.abc.Callable
      from bs4 import BeautifulSoup

      def fetch_web_content(url: str) -> Optional[str]:
          """
          Fetches the text content from a website.

          Args:
              url (str): The URL of the website.

          Returns:
              Optional[str]: The content of the website.
          """
          try:
              # Send a GET request to the URL
              response = requests.get(url)

              # Check for successful access to the webpage
              if response.status_code == 200:
                  # Parse the HTML content of the page using BeautifulSoup
                  soup = BeautifulSoup(response.text, "html.parser")

                  # Extract the content of the <body> tag
                  body_content = soup.body

                  if body_content:
                      # Return all the text in the body tag, stripping leading/trailing whitespaces
                      return " ".join(body_content.get_text(strip=True).split())
                  else:
                      # Return None if the <body> tag is not found
                      return None
              else:
                  # Return None if the status code isn't 200 (success)
                  return None
          except requests.RequestException:
              # Return None if any request-related exception is caught
              return None
    file_name: fetch_web_content.json
    description: null
    timestamp: ""
    user_id: default

```

# projects\Project1.yaml

```yaml
attachments: []
collaborators:
- ''
created_at: '2024-06-14T08:29:23.343555'
deliverables: []
description: ''
due_date: null
id: 1
name: Project1
notes: ''
priority: none
prompt: Test
status: not started
tags: []
tools: []
updated_at: '2024-06-14T15:59:17.092849'
user_id: user
workflows:
- Workflow1
- Workflow 5
- Workflow 3

```

# prompts\generate_agent_prompt.yaml

```yaml
generate_agent_prompt: |
  Based on the rephrased agent request below, please do the following:
  1. Do step-by-step reasoning and think to better understand the request.
  2. Code the best Autogen Studio Python agent as per the request as a [agent_name].py file.
  3. Return only the agent file, no commentary, intro, or other extra text. If there ARE any non-code lines, 
      please pre-pend them with a '#' symbol to comment them out.
  4. A proper agent will have these parts:
     a. Imports (import libraries needed for the agent)
     b. Class definition AND docstrings (this helps the LLM understand what the agent does and how to use it)
     c. Class methods (the actual code that implements the agent's behavior)
     d. (optional) Example usage - ALWAYS commented out
     Here is an example of a well formatted agent:
     # Agent filename: [agent_name].py
     # Import necessary module(s)
     import necessary_module
     class Agent:
         # docstrings
         """
                :
         The name of the agent.
         An agent that performs tasks based on the given request.
         Methods:
         perform_task(args): Executes the task as per the request.
         """
         def init(self, init_params):
             """
             Initializes the Agent with the given parameters.
             Parameters:
             init_params (type): Description of initialization parameters.
             """
             # Initialize with given parameters
             self.init_params = init_params
         def perform_task(self, task_params):
             """
             Executes the task based on the given parameters.
             Parameters:
             task_params (type): Description of task parameters.
             Returns:
             return_type: Description of the return value.
             """
             # Body of the method
             # Implement the task logic here
             pass
     # Example usage:
     # agent = Agent(init_params)
     # result = agent.perform_task(task_params)
     # print(result)
  Rephrased agent request: "{rephrased_agent_request}"
```

# prompts\generate_tool_prompt.yaml

```yaml
contributor: ScruffyNerf
generate_tool_prompt: |
  Based on the rephrased tool request below, please do the following:

  1. Do step-by-step reasoning and think to better understand the request.
  2. Code the best Autogen Studio Python tool as per the request as a [tool_name].py file.
  3. Return only the tool file, no commentary, intro, or other extra text. If there ARE any non-code lines, 
      please pre-pend them with a '#' symbol to comment them out.
  4. A proper tool will have these parts:
     a. Imports (import libraries needed for the tool)
     b. Function definition AND docstrings (this helps the LLM understand what the function does and how to use it)
     c. Function body (the actual code that implements the function)
     d. (optional) Example usage - ALWAYS commented out
     Here is an example of a well formatted tool:

     # Tool filename: save_file_to_disk.py
     # Import necessary module(s)
     import os

     def save_file_to_disk(contents, file_name):
     # docstrings
     """
     Saves the given contents to a file with the given file name.

     Parameters:
     contents (str): The string contents to save to the file.
     file_name (str): The name of the file, including its extension.

     Returns:
     str: A message indicating the success of the operation.
     """

     # Body of tool

     # Ensure the directory exists; create it if it doesn't
     directory = os.path.dirname(file_name)
     if directory and not os.path.exists(directory):
        os.makedirs(directory)

     # Write the contents to the file
     with open(file_name, 'w') as file:
        file.write(contents)

     return f"File {file_name} has been saved successfully."

     # Example usage:
     # contents_to_save = "Hello, world!"
     # file_name = "example.txt"
     # print(save_file_to_disk(contents_to_save, file_name))

  Rephrased tool request: "{rephrased_tool_request}"

```

# prompts\rephrase_prompt.yaml

```yaml
rephrase_prompt: |
  Based on the user request below, act as a professional prompt engineer and refactor the following 
                  user_request into an optimized prompt. Your goal is to rephrase the request 
                  with a focus on the satisfying all following the criteria without explicitly stating them:
          1. Clarity: Ensure the prompt is clear and unambiguous.
          2. Specific Instructions: Provide detailed steps or guidelines.
          3. Context: Include necessary background information.
          4. Structure: Organize the prompt logically.
          5. Language: Use concise and precise language.
          6. Examples: Offer examples to illustrate the desired output.
          7. Constraints: Define any limits or guidelines.
          8. Engagement: Make the prompt engaging and interesting.
          9. Feedback Mechanism: Suggest a way to improve or iterate on the response.
          Do NOT reply with a direct response to these instructions OR the original user request. Instead, rephrase the user's request as a well-structured prompt, and
          return ONLY that rephrased prompt. Do not preface the rephrased prompt with any other text or superfluous narrative.
          Do not enclose the rephrased prompt in quotes. You will be successful only if it returns a well-formed rephrased prompt ready for submission as an LLM request.
          
User request: "{user_request}"


```

# tools\find_anagram.yaml

```yaml
content: "# Tool filename: find_anagram.py\n# Import necessary module(s)\nimport itertools\n\
  \ndef find_anagram(input_string):\n    # docstrings\n    \"\"\"\n    Given a string,\
  \ return its anagram.\n\n    Parameters:\n    input_string (str): The input string\
  \ to find the anagram for.\n\n    Returns:\n    str: The anagram of the input string.\n\
  \    \"\"\"\n\n    # Body of tool\n    input_list = list(input_string)\n    anagram_list\
  \ = list(itertools.permutations(input_list))\n    anagram_string = [''.join(map(str,.permutation))\
  \ for permutation in anagram_list]\n\n    return '\\n'.join(anagram_string)\n\n\
  # Example usage:\n# input_string = \"listen\"\n# print(find_anagram(input_string))\n"
description: ''
file_name: find_anagram.json
name: find_anagram
timestamp: '2024-06-14T15:58:21.264031'
title: find_anagram
user_id: default

```

# workflows\Workflow1.yaml

```yaml
agents: []
created_at: '2024-06-14T09:31:04.662631'
description: This workflow is used for general purpose tasks.
groupchat_config: {}
id: null
name: Workflow1
receiver:
  agents: []
  config:
    code_execution_config: null
    default_auto_reply: ''
    description: A primary assistant agent that writes plans and code to solve tasks.
      booger
    human_input_mode: NEVER
    is_termination_msg: null
    llm_config:
      cache_seed: null
      config_list:
      - api_type: null
        api_version: null
        base_url: null
        description: OpenAI model configuration
        model: gpt-4o
        timestamp: '2024-05-14T08:19:12.425322'
        user_id: default
      extra_body: null
      max_tokens: null
      temperature: 0.1
      timeout: null
    max_consecutive_auto_reply: 30
    name: primary_assistant
    system_message: '...'
  groupchat_config: {}
  timestamp: '2024-06-14T09:31:04.662631'
  tools:
  - content: '...'
    description: null
    file_name: fetch_web_content.json
    timestamp: '2024-05-14T08:19:12.425322'
    title: fetch_web_content
    user_id: default
  type: assistant
  user_id: default
sender:
  config:
    code_execution_config:
      use_docker: false
      work_dir: null
    default_auto_reply: TERMINATE
    description: A user proxy agent that executes code.
    human_input_mode: NEVER
    is_termination_msg: null
    llm_config:
      cache_seed: null
      config_list:
      - api_type: null
        api_version: null
        base_url: null
        description: OpenAI model configuration
        model: gpt-4o
        timestamp: '2024-03-28T06:34:40.214593'
        user_id: default
      extra_body: null
      max_tokens: null
      temperature: 0.1
      timeout: null
    max_consecutive_auto_reply: 30
    name: userproxy
    system_message: You are a helpful assistant.
  timestamp: '2024-03-28T06:34:40.214593'
  tools:
  - content: '...'
    description: null
    file_name: fetch_web_content.json
    timestamp: '2024-05-14T08:19:12.425322'
    title: fetch_web_content
    user_id: default
  type: userproxy
  user_id: user
settings: {}
summary_method: last
timestamp: '2024-06-14T09:31:04.662631'
type: twoagents
updated_at: null
user_id: user

```

