# PowerShell tips



1. # 字符串拼接

​      this is an excellent place to use a string formatter instead of losing your sanity over escaping and interpolation. Second, instead of `Start-Process` you should be able to use the `&` statement:

```
$vhdPath = '\\Computer1\ServerBackups$\{0}\{1}.vhd' -f $ComputerName, $JobTitle
& C:\Powershell-Backup\disk2vhd.exe C: $vhdPath
```

or If it must be a oneliner

```
& C:\Powershell-Backup\disk2vhd.exe C: ('\\Computer1\ServerBackups$\{0}\{1}.vhd' -f $ComputerName, $JobTitle)
```



```powershell
'{0} {1}' -f "A","B"
```

```powershell
$username = "Tom"
$age = 27
$stringMsg = "Hello, my name is $($username), my age is $($age)"
```

```powershell
 echo "$(hostname)"
```



```powershell
"It is now {0:yyyy-MM-dd hh:mm:ss}" -f (Get-Date)
```

```powershell
Write-Host ("It is now {0}" -f (Get-Date))
```



## 压缩：

```powershell
.\autorunsc64.exe  -a * -ct -nobanner > "$(hostname)-1.csv"
```



```powershell
Compress-Archive -Path .\WK1-1.csv -DestinationPath WK1-1.zip -CompressionLevel Fastest
```

## Send JSON File



Linux:

```shell
curl -X POST http://localhost:8000/  -H "Content-Type: application/json" -d @sendfile.json
```

Windows 10:

```shell
curl -X POST http://127.0.0.1:8000  -H "Content-Type: application/json" -d '@sendfile.json'
```

```shell
curl -X POST http://127.0.0.1:8000  -H "Content-Type: application/json" -d '{"Ticks":1000}'
```

upload file with POST

```shell
curl -X POST http://127.0.0.1:8000  -F "file=@WK1-1.zip"
```



## Flask API:

```python


import os
from flask import Flask, flash, render_template, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = '/home/ubuntu/uploads'
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

@app.route('/') 
def main(): 
	return render_template("index.html") 

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
           
@app.route('/', methods=['GET', 'POST','PUT'])
def upload_file():
    # if request.method == 'PUT':
    #     return f"send: {request.json['Minutes']}"
    
    if request.method == 'GET':
        return "GET method"
    
    # if request.method == 'POST':
    #     json_data = request.json
    #     ret = json_data.get('TotalMicroseconds')
    #     return str(ret + 100)
    if request.method == 'POST':
        file = request.files['file']
        filename = secure_filename(file.filename)
        print(filename)
        print(app.config['UPLOAD_FOLDER'])
        file.save(os.path.join(app.config['UPLOAD_FOLDER'],filename))
        
        return "OK"

if __name__ == '__main__': 
	app.run(host='0.0.0.0',port=8000,debug=True)

```



