# Intro to Azure DevOps REST API

Team Foundation Services and Azure DevOps are fantastic tools with lots of features out-of-the-box. If that's not enough they also offer a comprehensive REST API that gives you access to a world of possibilities. In this article you are going to learn how to get started with the TFS/Azure DevOps REST API.

## Our demo

To demonstrate how you can access the Azure DevOps API we are going to write simple html page that will display the projects your Azure DevOps account has access to. 

Microsoft provides several client and server libraries to interact with the Azure DevOps REST API but for this example we are going to access the API directly with some plain-vanilla JavaScript. 

## Authentication

TFS/Azure DevOps offer different possibilities to authenticate a client. The easiest is a personal access token (PAT). A PAT is a code linked to our account that guarantees access to all or part of the Azure DevOps REST API. PATs have an expiration date and they can be revoked at any time.

To obtain a PAT follow [these instructions](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=current-page). Basically, somewhere within your profile you should be able to create a PAT with the permissions you require.

Be careful not to share you PATs. They give access to your repositories to whoever knows them.

## REST API endpoints

[Microsoft's documentation](https://docs.microsoft.com/en-us/rest/api/azure/devops/core/projects/list?view=azure-devops-rest-5.1) shows that to access our project list we need to use the following end-point:

``GET https://dev.azure.com/{organization}/_apis/projects?api-version=5.1``

and we can expect it to return a JSON object looking like this:

```json
{
  "count": 2,
  "value": [
    {
      "id": "eb6e4656-77fc-42a1-9181-4c6d8e9da5d1",
      "name": "Fabrikam-Fiber-TFVC",
      "description": "Team Foundation Version Control projects.",
      "url": "https://dev.azure.com/fabrikam/_apis/projects/eb6e4656-77fc-42a1-9181-4c6d8e9da5d1",
      "state": "wellFormed"
    },
    {
      "id": "6ce954b1-ce1f-45d1-b94d-e6bf2464ba2c",
      "name": "Fabrikam-Fiber-Git",
      "description": "Git projects",
      "url": "https://dev.azure.com/fabrikam/_apis/projects/6ce954b1-ce1f-45d1-b94d-e6bf2464ba2c",
      "state": "wellFormed"
    }
  ]
}
```

Endpoints are different depending of the TFS/Azure DevOps version. The URL for TFS will look like this:

``GET https://{instance}/{collection}/_apis/projects?api-version=4.1``

Where {instance} is your on-premises TFS server.



## Getting down to business

Our page contains a form to gather the parameters we need for our API call: PAT and organization name.

```html
<div>
  <input id="pat" type="password" placeholder="Personal Access Token"/><br/>
  <input id="organization" type="text" placeholder="Organization"/><br/>
  <button id="go" type="button" onclick="load_projects()">Load</button>
</div>
```

We then add some placeholders for our project list and/or any possible errors we may run into.

```html
<div>
  <p>Projects:</p>
  <ul id="projects">
  </ul>
  <div id="errors"></div>
</div>
```

When you click the *Load* button the browser will call the *load_projects()* function. *load_projects()* will fetch the list of projects and add items to our list.

In order to get our request authorized we first need to sort out the request headers.

```javascript
function buildHeaders() {
  // Get PAT and use it to build our Authorization header
  let pat = document.querySelector("#pat").value;
  let authHeader = "Basic " + btoa(pat + ":" + pat);
  return new Headers({'Authorization': authHeader});
}
```    

Then we determine the URL for our end-point.

```javascript
function buildUrl() {
  let organization = document.querySelector("#organization").value;
  return `https://dev.azure.com/${organization}/_apis/projects?api-version=5.1`;
}
```

And we can finally request the data. Here we are using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), providing one (inline) function to handle a valid response and parse the response's body as a JSON object and a second one to handle an error. The *processProjects* function will be called once the JSON object has been parsed without errors.

```javascript
function load_projects() {
  let headers = buildHeaders();
  let url = buildUrl();

  fetch(url, {method: 'GET', headers: headers})
    .then(function (response) {
        return response.json().then(processProjects); 
      })
    .catch(displayError);
}
```

The error handler writes the error text inside the error placeholder.

```javascript
function displayError(err) {
    document.querySelector("#errors").innerHTML = err;
}
```

We finally can process the JSON object returned by Azure DevOps. For our end-point the JSON object is expected to contain 2 fields: `count` and `value`. `count` is the number of items in `value` and `value` is an array that in our case contains [TeamProjectReference](https://docs.microsoft.com/en-us/rest/api/azure/devops/core/projects/list?view=azure-devops-rest-5.1#teamprojectreference) objects. 

```javascript
function processProjects(data) {
    if (data.count > 0) {
        let ul = document.querySelector("#projects");
        data.value.forEach(projectRef => {
            let li = document.createElement("li");
            li.innerText = projectRef.name;
            ul.appendChild(li);
        });
    }
}
```

That was it. You can now go thru the Azure DevOps REST API and start creating crazy apps. No pressure, just relax and watch it happen. As for me, I am a C#/Java developer and JavaScript sucks. Next, I would like to interact with this REST API with a more sensible language, like TypeScript, but that will be another article.

