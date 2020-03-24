# up42-py
Python API interface for using UP42

![coverage](coverage.svg)

Documentation with Quickstart, Tutorials & Code-Reference: [https://up42.github.io/up42-py/](https://up42.github.io/up42-py/)


## API structure:
See also [docs/quickstart](https://up42.github.io/up42-py/quickstart/01_quickstart)

- The Python Api uses seven object classes, representing the **hierachical structure** of UP42: **Api > Project > Workflow > Job > JobTask** and **Catalog & Tools**.
- Each object provides the full functionality at that specific level and can spawn elements of one level below, e.g.
    - `workflow = Project().create_workflow()`
    - `job = workflow.create_and_run_job()`
<br>
- Usually the user starts with the *Api* object, then spawns objects of a lower level (e.g. initializes a project, creates a new workflow, runs a job etc.).
- In some cases you might want to access a lower level object directly, e.g. a job that was already run on UP42. Then you can can directly initiate that job object via its job-id.


## Installation
See also [docs/installation](https://up42.github.io/up42-py/installation/)

1. **Optional**: Create a virtual environment:
```bash
mkvirtualenv --python=$(which python3.7) up42-py
```

2. Install locally with systemlink (code changes are reflected):
```bash
git clone git@github.com:up42/up42-py.git
cd up42-py
pip install -r requirements.txt
pip install -e .
```

3. Create a new project on [UP42](https://up42.com).

4. Create a `config.json` file and fill in the [project credentials](https://docs.up42.com/getting-started/first-api-request.html#run-your-first-job-via-the-api).
```json
{
  "project_id": "...",
  "project_api_key": "..."
}
```

4. Test it in Python! This will authenticate with the UP42 Server and get the project information.
```python
import up42

api = up42.Api(cfg_file="config.json")
project = api.initialize_project()
print(project)
```

## Quickstart

### Api structure:

- The UP42 Python Api uses seven object classes, representing the **hierachical structure** of UP42: **Api > Project > Workflow > Job > JobTask** and **Catalog & Tools**.
- Each object provides the full functionality at that specific level and can spawn elements of one level below, e.g.
    - `workflow = Project().create_workflow()``
    - `job = workflow.create_and_run_job()``
<br> 
- Usually the user starts with the *Api* object, then spawns objects of a lower level (e.g. initializes a project, creates a new workflow, runs a job etc.). 
- To access a lower-level object directly, e.g. a job that was already run on UP42 initialize the object directly via `api.initialize_job(job_id='123456789')`.

### 30 sec example:

See also [docs/30-seconds-example](https://up42.github.io/up42-py/quickstart/01_quickstart/#30-seconds-example) or the Jupyter Notebook in the examples folder.

```python
import up42

# Authenticate: Gets the the project credentials saved in the config.json file.
api = up42.Api(cfg_file="config.json")
project = api.initialize_project()

# Create a workflow & add blocks/tasks to the workflow.
workflow = project.create_workflow(name="30-seconds-workflow", use_existing=True)
blocks = api.get_blocks(basic=True)
input_tasks= [blocks['sobloo-s2-l1c-aoiclipped'],
              blocks['sharpening']]
workflow.add_workflow_tasks(input_tasks=input_tasks)

# Define the aoi and input parameters of the workflow to run it.
aoi = workflow.read_vector_file("data/aoi_berlin.geojson", as_dataframe=True)
input_parameters = workflow.construct_parameter(geometry=aoi,
                                                geometry_operation="bbox",
                                                start_date="2019-04-01",
                                                end_date="2019-05-30",
                                                limit=1)
print(input_parameters)

# Run the workflow as a job
job = workflow.create_and_run_job(input_parameters=input_parameters)
job.track_status()

# Plot the scene quicklooks.
job.download_quicklook()
job.plot_quicklook()

# Plot & analyse the results.
results_fp = job.download_result()
print(results_fp)

job.plot_result()

job.map_result()
```
