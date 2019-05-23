# My Project Product Rest API

The folder structure will look like this:
```
~/environment/myproject-product-restapi
├── app.py
├── custom_logger.py
├── product_routes.py
├── products.json
├── requirements.txt
├── venv/
├── Dockerfile
├── README.md
└── .gitignore
```

## Step 1: Create Backend using Python Flask REST API
- Create basic CRUD functionality for a product-management service
- The Service will be triggered by developers via a web api
```
| HTTP METHOD | URI                                     | ACTION                      |
|-------------|-----------------------------------------|-----------------------------|
| GET         | http://[hostname]/products              | Gets all products           |
| GET         | http://[hostname]/products/<product_id> | Gets one product            |
| POST        | http://[hostname]/products              | Creates a new product       |
| PUT         | http://[hostname]/products/<product_id> | Updates an existing product |
| DELETE      | http://[hostname]/products/<product_id> | Deletes a product           |
```

### Step 1.1: Create a CodeCommit Repository
```
$ aws codecommit create-repository --repository-name myproject-product-restapi
```

### Step 1.2: Clone the repository
```
$ cd ~/environment
$ git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/myproject-product-restapi
```

### Step 1.3: Set up .gitignore
```
$ cd ~/environment/myproject-product-restapi
$ vi .gitignore
```
```
# Byte-compiled / optimized / DLL files
__pycache__/
.api/__pycache__/
.api/products/__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
.hypothesis/
.pytest_cache/

# Flask stuff:
instance/
.webassets-cache

# Virtual environment
venv
*.pyc

# IDE
.vscode
```

### Step 1.4: Test access to repo by adding README.md file and push to remote repository

```bash
$ cd ~/environment/myproject-product-restapi
$ echo "myproject-product-restapi" >> README.md
$ git add .
$ git commit -m "Adding README.md"
$ git push origin master
```

### Step 1.5: Navigate to working directory
```bash
$ cd ~/environment/myproject-product-restapi
$ python3 -m venv venv
$ source venv/bin/activate
$ venv/bin/pip install flask
$ venv/bin/pip install flask-cors
```

###  Step 1.6: Prepare static database
```bash
$ cd ~/environment/myproject-product-restapi
$ vi products.json
```
```json
[
  {
    "product_id": "4e53920c-505a-4a90-a694-b9300791f0ae",
    "name": "Soap",
    "description": "Used to wash body",
    "image_url": "https://via.placeholder.com/150"
  },
  {
    "product_id": "2b473002-36f8-4b87-954e-9a377e0ccbec",
    "name": "Shampoo",
    "description": "Used to wash hair",
    "image_url": "https://via.placeholder.com/150"
  },
  {
    "product_id": "3f0f196c-4a7b-43af-9e29-6522a715342d",
    "name": "Tissue",
    "description": "thin, soft paper, typically used for wrapping or protecting fragile or delicate articles.",
    "image_url": "https://via.placeholder.com/150"
  }
]
```

### Step 1.7: Add product_routes.py
```
$ cd ~/environment/myproject-product-restapi
$ vi product_routes.py
```
```python
import os
import uuid
from flask import Blueprint
from flask import Flask, json, Response, request
from custom_logger import setup_logger

# set up the custom logger and the Blueprint
logger = setup_logger(__name__)
product_module = Blueprint('products', __name__)

logger.info("Intialized product routes")

THIS_FOLDER = os.path.dirname(os.path.abspath(__file__))
my_file = os.path.join(THIS_FOLDER, 'products.json')

# load products static db from json file
with open(my_file) as f:
    products = json.load(f)

# allow the default route to return a health check
@product_module.route('/')
def health_check():
    return "This a health check. Product Management Service is up and running."

@product_module.route('/products')
def get_all_products():

    #returns all the products coming from dynamodb
    serviceResponse = json.dumps({'products': products})
    
    resp = Response(serviceResponse)
    resp.headers["Content-Type"] = "application/json"
    
    return resp
    
@product_module.route("/products/<product_id>", methods=['GET'])
def get_product(product_id):

    #returns a product given its id
    serviceResponse = json.dumps({'products': products[0]})

    resp = Response(serviceResponse)
    resp.headers["Content-Type"] = "application/json"

    return resp

@product_module.route("/products", methods=['POST'])
def create_product():

    #creates a new product. The product id is automatically generated.
    product_dict = json.loads(request.data)

    product = {
        'product_id': str(uuid.uuid4()),
        'name': product_dict['name'],
        'description': product_dict['description'],
        'image_url': product_dict['image_url']
    }

    serviceResponse = json.dumps({'products': product})
    resp = Response(serviceResponse)

    resp.headers["Content-Type"] = "application/json"

    return resp

@product_module.route("/products/<product_id>", methods=['PUT'])
def update_product(product_id):
    
    #updates am existing product 
    product_dict = json.loads(request.data)

    products[0]['name'] = request.json.get('name', products[0]['name'])
    products[0]['description'] = request.json.get('description', products[0]['description'])
    products[0]['image_url'] = request.json.get('image_url', products[0]['image_url'])
    
    serviceResponse = json.dumps({'products': products[0]})

    resp = Response(serviceResponse)
    resp.headers["Content-Type"] = "application/json"

    return resp

@product_module.route("/products/<product_id>", methods=['DELETE'])
def delete_product(product_id):
    
    #deletes a product given its id.
    serviceResponse = json.dumps({"products" : "Deletes a product with id: {}".format(product_id)})

    products.remove(products[0])

    resp = Response(serviceResponse)
    resp.headers["Content-Type"] = "application/json"

    return resp
```

### Step 1.8: Add app.py and custom_logger.py
- Add app.py
```
$ cd ~/environment/myproject-product-restapi
$ vi app.py
```
```python
from flask import Flask
from flask_cors import CORS

# Add new blueprints here
from product_routes import product_module

# Initialize the flask application
app = Flask(__name__)
CORS(app)

# Add a blueprint for the products module
app.register_blueprint(product_module)

# Run the application
app.run(host="0.0.0.0", port=5000, debug=True)
```

- custom_logger.py
```
$ cd ~/environment/myproject-product-restapi
$ vi custom_logger.py
```
```python
import logging

def setup_logger(name):
    formatter = logging.Formatter(fmt='[%(levelname)s][{}] %(asctime)s:%(threadName)s:%(message)s'.format(name), datefmt='%Y-%m-%d %H:%M:%S')

    handler = logging.StreamHandler()
    handler.setFormatter(formatter)

    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)
    logger.addHandler(handler)
    
    return logger
```

### Step 1.9: Run Locally and Test
```bash
$ cd ~/environment/myproject-product-restapi
$ python app.py
$ curl http://localhost:5000
```

### (TODO) Step 1.10: Backend Unit Tests

### Step 1.11: Generate requirements.txt
```
$ cd ~/environment/myproject-product-restapi
$ pip freeze > requirements.txt
```

### Step 1.12: Create the Dockerfile
```bash
$ cd ~/environment/myproject-product-restapi 
$ vi Dockerfile
```
```
# Set base image to python
FROM python:3.7
ENV PYTHONBUFFERED 1

# Copy source file and python req's
COPY . /app
WORKDIR /app

# Install requirements
RUN pip install --upgrade pip
RUN pip install -r requirements.txt

# Set image's main command and run the command within the container
EXPOSE 5000
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### Step 1.13: Build, Tag and Run the Docker Image locally
Replace:
- AccountId: 707538076348
- Region: us-east-1

```bash
$ docker build -t myproject-product-restapi .
$ docker tag myproject-product-restapi:latest 707538076348.dkr.ecr.us-east-1.amazonaws.com/myproject-product-restapi:latest
$ docker run -p 5000:5000 myproject-product-restapi:latest
```

### Step 1.14: Test CRUD Operations
- Test Get all Products
```
curl -X GET \
  http://localhost:5000/products \
  -H 'Host: localhost:5000'
```

- Test Get Product
```
curl -X GET \
  http://localhost:5000/products/d58ada00-1d53-4164-9453-b8fe3fb080c5 \
  -H 'Host: localhost:5000' 
```

- Test Create Product
```
curl -X POST \
  http://localhost:5000/products \
  -H 'Content-Type: application/json' \
  -d '{
  "name":"Product G",
  "description": "Nulla nec dolor a ipsum viverra tincidunt eleifend id orci. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.",
  "image_url": "https://via.placeholder.com/200"
}'
```

- Test Update Product
```
curl -X PUT \
  http://localhost:5000/products/d58ada00-1d53-4164-9453-b8fe3fb080c5 \
  -H 'Content-Type: application/json' \
  -d '{
  "name":"egg 123",
  "description": "my working description dasdasds",
  "image_url": "product_image testes update test"
}'
```

- Test Delete Product
```
curl -X DELETE \
  http://localhost:5000/products/3f0f196c-4a7b-43af-9e29-6522a715342d \
  -H 'Content-Type: application/json' 
```

### Step 1.15: Push to Remote Repository
```
$ cd ~/environment/myproject-product-restapi
$ git add .
$ git commit -m "Initial Commit"
$ git push origin master
```

### Step 1.16: Create the ECR Repository
```
$ aws ecr create-repository --repository-name myproject-product-restapi
```

### Step 1.17: Run login command to retrieve credentials for our Docker client and then automatically execute it (include the full command including the $ below).
```
$ $(aws ecr get-login --no-include-email)
```

### Step 1.18: Push our Docker Image
```
$ docker push 707538076348.dkr.ecr.us-east-1.amazonaws.com/myproject-product-restapi:latest
```

### Step 1.19: Validate Image has been pushed
```
$ aws ecr describe-images --repository-name myproject-product-restapi
```

### (Optional) Clean up
```
$ aws ecr delete-repository --repository-name myproject-product-restapi --force
$ aws codecommit delete-repository --repository-name myproject-product-restapi
$ rm -rf ~/environment/myproject-product-restapi
```
