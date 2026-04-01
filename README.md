# From Experiments to Workflows: Use Cases

This repository contains the test models, generated workflows, and setup instructions
for the research project *"From Experiments to Workflows"*.

The project extends the [LEQO low-code backend](https://github.com/LEQO-Framework/leqo-backend)
with workflow generation capabilities, supporting three types of models:
fixed-value models, placeholder models, and plugin models.

## Repository Structure

```
.
├── models/
│   ├── fixed-value-model.json      # Test model without placeholders
│   ├── placeholder-model.json      # Test model with a placeholder value
│   └── plugin-model.json           # Test model with Classical K-Means plugin
├── workflows/
│   ├── fixed-value-workflow.bpmn   # Generated workflow for fixed-value model
│   ├── placeholder-workflow.bpmn   # Generated workflow for placeholder model
│   ├── plugin-workflow.bpmn        # Generated workflow for plugin model
│   └── qrms/
│       └── qrms.zip                # Generated QRMs for quantum groups
└── README.md
```

## Prerequisites

- [Docker](https://www.docker.com/) and Docker Compose
- [Node.js](https://nodejs.org/) (for the low-code modeler and backend)
- A web browser

## Setup

The setup consists of four groups of services that need to be started separately.

### Step 1: Start the QuantME environment (Camunda, Winery, etc.)

Clone and start the QuantME-UseCases Docker environment:

```bash
git clone -b fix/2025-icse-docker https://github.com/UST-QuAntiL/QuantME-UseCases.git
cd QuantME-UseCases/2025-icse/docker
docker-compose pull
docker-compose up -d
```

This starts the following relevant services:

| Service            | URL                        | Description                              |
|--------------------|----------------------------|------------------------------------------|
| Workflow Modeler   | http://localhost:1893      | QuantME workflow editor                  |
| Camunda Engine     | http://localhost:8090      | Workflow engine for executing workflows  |
| Winery             | http://localhost:8093      | TOSCA service template repository        |

Wait until all services are running before proceeding.

### Step 2: Start Qunicorn

Clone the Qunicorn repository and replace the default `docker-compose.yml` with the
provided configuration:

```bash
git clone https://github.com/qunicorn/qunicorn-core.git
cd qunicorn-core
```

Replace the `docker-compose.yml` with the version from this repository or from
`daria_docs/docker-compose(2).yaml`, then start the services:

```bash
docker-compose pull
docker-compose up -d
```

| Service          | URL                        | Description                                    |
|------------------|----------------------------|------------------------------------------------|
| Qunicorn Server  | http://localhost:8080      | Quantum circuit execution middleware           |
| Qunicorn Worker  | (internal)                 | Background worker for quantum job execution    |
| Keycloak         | http://localhost:8041      | Authentication service                         |

### Step 3: Start the low-code modeler and backend

Clone and start the low-code modeler:

```bash
git clone https://github.com/LEQO-Framework/low-code-modeler.git
cd low-code-modeler
npm install
npm run dev
```

In a separate terminal, clone and start the low-code backend:

```bash
git clone https://github.com/LEQO-Framework/leqo-backend.git
cd leqo-backend
# Follow the setup instructions in the repository's README
```

| Service            | URL                        | Description                                      |
|--------------------|----------------------------|--------------------------------------------------|
| Low-Code Modeler   | http://localhost:4200      | Visual editor for quantum low-code models        |
| Low-Code Backend   | http://localhost:8000      | Transformation pipeline and workflow generation  |

### Step 4: Start the QHAna environment (only for plugin models)

This step is only required for testing models with ML nodes (e.g., Classical K-Means).

```bash
git clone https://github.com/UST-QuAntiL/qhana-docker.git
cd qhana-docker
docker-compose pull
docker-compose up -d
```

| Service             | URL                        | Description                                  |
|---------------------|----------------------------|----------------------------------------------|
| QHAna Plugin Runner | http://localhost:5005      | Executes ML plugins (e.g., Classical K-Means)|

## Usage

### Loading a test model

1. Open the low-code modeler at http://localhost:4200.
2. Import one of the test models from the `models/` directory.

The following three models are provided:

**Fixed-Value Model:**
A qubit node, a register node, a Hadamard gate, and a Measurement node connected
sequentially. All values are specified directly. This model generates a standard
workflow that compiles the circuit at generation time and executes it via Qunicorn.

**Placeholder Model:**
A Number node, a Basis Encoding node, and a Measurement node connected sequentially.
The Number node's value is set to a placeholder variable instead of a concrete number.
This model generates a placeholder workflow where the user provides the value at runtime
through a Camunda form field.

**Plugin Model:**
A File node, a Number node, and a Classical Clustering (K-Means) plugin node.
The File node provides the entity points URL, the Number node specifies the number of
clusters. This model generates a plugin workflow that calls the QHAna plugin runner.

### Generating the workflow

1. Click the **Compile** button in the modeler with the compilation target set to
   **workflow**.
2. Wait for the backend to generate the workflow. The status can be tracked in the
   modeler's history view.
3. Once the status is **completed**, download the generated workflow and the QRM
   archive (if the model contains quantum elements).

Alternatively, you can use the pre-generated workflows from the `workflows/` directory.

### Deploying and executing in Camunda

1. Open the Camunda web interface at http://localhost:8090.
2. Log in with the credentials **demo / demo**.
3. Deploy the generated workflow file.
4. Start a new process instance and fill in the required form fields:
   - **IP Address**: The IP address of the host machine (e.g., `localhost`)
   - **Qunicorn Port**: `8080`
   - **Backend Port**: `8000`
   - **Plugin Port**: `5005` (only for plugin models)
   - **Placeholder** (only for placeholder models): The value to substitute
5. Open the **Cockpit** to monitor the running process instance.
6. The process token should advance through the tasks and reach the
   **Analyze Results** user task.

## References

- [LEQO Low-Code Modeler](https://github.com/LEQO-Framework/low-code-modeler)
- [LEQO Low-Code Backend](https://github.com/LEQO-Framework/leqo-backend)
- [Qunicorn](https://github.com/qunicorn/qunicorn-core)
- [QuantME-UseCases](https://github.com/UST-QuAntiL/QuantME-UseCases)
- [QHAna Docker](https://github.com/UST-QuAntiL/qhana-docker)
