# jq基础

## Object constructors：



```powershell
PS C:\Users\sheng> get-Process | Select-Object Name,Id | ConvertTo-Json | jq -r '{"first":.[0],"last":.[-1]}'
{
  "first": {
    "Name": "AdobeCollabSync",
    "Id": 5300
  },
  "last": {
    "Name": "yundetectservice",
    "Id": 16112
  }
}
```



sample：

```powershell
get-Process | Select-Object Name,Id | ConvertTo-Json | jq -r '[ .[] | {Name,Id} ]'
```

map() sort_by() reverse length



echo '[{"a":"va1"},{"a":"va2"}]' | jq -r '.[].a'

![image-20231118085933272](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231118085933272.png)

![image-20231118085936991](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231118085936991.png)

![image-20231118085943816](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231118085943816.png)

```shell
Filter:

if .[].Name == "Host" then "X" elif .[].Id == 1 then "O" else "Many" end

JSON:

[
 {
    "Name": "Host",
    "Id": 1
 },
 {
    "Name": "WUDF",
    "Id": 2
 }
]
```





![image-20231117194352190](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117194352190.png)

![image-20231117194511479](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117194511479.png)

![image-20231117194556785](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117194556785.png)



![image-20231117194701847](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117194701847.png)

toString

![image-20231117200332239](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200332239.png)

![image-20231117200516667](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200516667.png)



![image-20231117200546480](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200546480.png)





![image-20231117200621423](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200621423.png)



![image-20231117200831479](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200831479.png)

![image-20231117200921345](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117200921345.png)

![image-20231117201025871](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117201025871.png)

![image-20231117201550447](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117201550447.png)

![image-20231117201622676](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117201622676.png)

![image-20231117201712921](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117201712921.png)

![image-20231117202133246](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117202133246.png)

![image-20231117203209044](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117203209044.png)

tonumber

![image-20231117203351105](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117203351105.png)

![image-20231117203735975](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231117203735975.png)











