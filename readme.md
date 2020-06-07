# Turnitin-API
An unofficial REST API for Turnitin.

## Endpoints

All endpoints are relative to [https://turnitin-api.herokuapp.com](https://turnitin-api.herokuapp.com).

| URL | Method | Data | Response |
|:----|:-------|:-----|:---------|
| `/login` | `POST` | `{ email: "email", password: "password" }` | `{ auth: [json] }` |
| `/classes` | `POST` | `{ auth: [json] } ` | `[{title: "title", url: "url"}, ...]` |
| `/assignments` | `POST` | `{ auth: [json], url: "url" }` | `{ title: "title", dates: { due: "due", post: "post", start: "start" }, info: "info", submission: "url" }` |
| `/download` | `POST` | `{ auth: [json], assignment: [json], pdf: [true/false] } ` | Raw bytes of file submitted |

## Examples
JavaScript:
```javascript
var email = "email@example.com"
var password = "password";
var host = "https://turnitin-api.herokuapp.com";
var authKeys;
var toDownload;
var pdf = false;
fetch(host + "/login", {
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    "body": JSON.stringify({email: email, password: password}),
    "method": "POST"
}).then(response=>{
    return response.json();
}).then(json=>{
    authKeys = json.auth;
}).then(()=>{
    return fetch(host + "/classes", {
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        },
        "body": JSON.stringify({auth: authKeys}),
        "method": "POST"
    });
}).then(response=>{
    return response.json();
}).then(classes=>{
    console.log(classes);
    return classes[0];
}).then(firstClass=>{
    return fetch(host + "/assignments", {
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        },
        "body": JSON.stringify({auth: authKeys, url: firstClass.url}),
        "method": "POST"
    });
}).then(response=>{
    return response.json();
}).then(assignments=>{
    console.log(assignments);
    return assignments[0];
}).then(assignment=>{
    toDownload = assignment;
    return fetch(host + "/download", {
        headers: {
            'Content-Type': 'application/json'
        },
        "body": JSON.stringify({auth: authKeys, assignment: toDownload, pdf: pdf}),
        "method": "POST"
    });
}).then(response=>{
    return response.blob();
}).then(blob=>{
    console.log(blob);
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    var filename = pdf?(toDownload.title+".pdf"):toDownload.file;
    link.setAttribute('download', filename);
    document.body.appendChild(link);
    link.click();
});
```
Python 3:
```python
import requests
import json

url = "https://turnitin-api.herokuapp.com"
USERNAME = "email@example.com"
PASSWORD = "password"

with requests.Session() as s:
    login_result = s.post(url + "/login", json={
        "email": USERNAME,
        "password": PASSWORD
    })
    auth = json.loads(login_result.text)

    classes_result = s.post(url + "/classes", json=auth)
    classes = json.loads(classes_result.text)
    print(classes[0])

    first_class_data = dict(auth)
    first_class_data["url"] = classes[0]["url"]
    assignments_result = s.post(url + "/assignments", json=first_class_data)
    assignments = json.loads(assignments_result.text)
    print(assignments[0])
```
