# MS-Forms-Questions-Download
This will provide a list of every MS Forms question you've made. 
Useful if you don't have Azure permissions to give yourself API credentials and don't want to spend HOURS trying to do it via Azure-AD module commands.

# Step One: Getting a list of your forms
Make sure you're authenticated in a browser with MS already and go to https://forms.office.com/formapi/api/forms
Note the ownerId and ownerTenantId (Control F to find them easily - they'll be the same for every form)
Save this page somewhere

# Step Two: Generate a list of URLs to your forms

Replace the ownerTenantId and the ownerId with the values you noted before

```
$jsonFilePath = "PATH_TO_YOUR_JSON" #for example "C:\Temp\forms.json"
$json = Get-Content -Path $jsonFilePath -Raw
$object = $json | ConvertFrom-Json
$quizid = $object[0].value

foreach ($i in $quizid){
"https://forms.office.com/formapi/api/ownerTenantId/users/ownerId/forms('" + $i.id + "')/questions"
}
```

# Step Three: Downloading all of the forms
You'll now have a list of URLS paste those URLS into the const array

Go back to your browser which you are authenticated with MS. Change the browswer settings to automatically download files to a folder. I chose C:\Temp\jsonDownloads
Run the following code (with your list of URLs in the const array)
```
const urls = [
"https://forms.office.com/formapi/api/ownerTenantId/users/ownerId/forms('formId')/questions",
"https://forms.office.com/formapi/api/ownerTenantId/users/ownerId/forms('formId')/questions"
];

async function fetchAndSaveWithDelay(url, index) {
  try {
    const response = await fetch(url);
    const html = await response.text();

    const jsonData = { url, html };

    // Convert JavaScript object to JSON string
    const jsonString = JSON.stringify(jsonData, null, 2);

    // Save JSON string to a file
    const fileName = `${index + 1}.json`;
    // Assuming you are working in a browser environment
    const blob = new Blob([jsonString], { type: 'application/json' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = fileName;
    link.click();

    console.log(`Page ${index + 1} saved as ${fileName}`);

  } catch (error) {
    console.error(`Error fetching page ${index + 1}:`, error);
  }
}

// Function to loop through the list of URLs with a delay
async function fetchPagesWithDelay() {
  for (let i = 0; i < urls.length; i++) {
    await fetchAndSaveWithDelay(urls[i], i);
    // Wait for half a second (500 milliseconds) before the next iteration
    await new Promise(resolve => setTimeout(resolve, 500));
  }
}

// Start fetching pages with a delay
fetchPagesWithDelay();
```

# Step Four: Grabbing the questions from each form
You should now hopefully a JSON file per quiz/form you have created.
Back in Powershell, run the following

```
$files = Get-ChildItem "PATH_TO_YOUR_DOWNLOADED_FILES" -Filter *.json

$qList = @()
foreach ($file in $files){

$jsonFilePath = $file.FullName
$json = Get-Content -Path $jsonFilePath -Raw
$jsonObject = $json | ConvertFrom-Json
$htmlObject = $jsonObject.html | ConvertFrom-Json

    foreach ($q in $htmlObject.value.title)
    {
     $qList += $q
    }
}

$filePath = "C:\Temp\Questions.txt"
$qList | Out-File -FilePath $filePath -Encoding UTF8 -Append
```

