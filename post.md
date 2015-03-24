## Moving Glance Image between Projects

In this post I describe a simple and efficent mechanism to move an image between two different Open Stack Projects/Tenants in case you are not an admin.

Assume you have:

* an OpenStack (OS) project called PROJECT1
* an OS project called PROJECT2
* a glance image in PROJECT1 called IMAGE1 with id = 123

```
(this command is issued on Terminal1 with PROJECT1 credentials)
# glance image-list 
+---------+-------------+-------------+------------------+--------------+--------+
| ID      | Name        | Disk Format | Container Format | Size         | Status |
+---------+-------------+-------------+------------------+--------------+--------+
| 123     | IMAGE1      | raw         | bare             | 2147483648   | active |
| 456     | debian      | raw         | bare             | 2147483648   | active |
| 789     | ubuntu 14   | raw         | bare             | 2147483648   | active |
| 101     | your distr  | raw         | bare             | 2147483648   | active |
+---------+-------------+-------------+------------------+--------------+--------+
````

Assume now you are not the OS admin and the two projetcs do not share any resource.

![Moving Glance Image between Projects](http://oi61.tinypic.com/wim4h1.jpg)

Let's now see how to copy IMAGE1 from PROJECT1 to PROJECT2

* from your PC or from a remote PC or from an instance on OS open two terminal windows (Terminal 1 and Terminal 2)
* from within Terminal 1 issue the following command 

`mkfif glance_fifo`

This creates a named pipe which will be used to transfer the image
* from within Terminal 1 issue the following command
* 
`glance image-download --progress --file glance_fifo 123`

* from within Terminal 2 issue the following command

`glance image-create --name <YOUR IMAGE NAME> --file glance_fifo --disk-format raw --container-format bare`

**NOTE: I have omitted the `--progress` in the above command on purpose, as the version of python-glanceclient I am using has a bug. Newer version should fix it**

You should now see a progress bar like that `[================> ] 55%` in Terminal 1

This is showing you that IMAGE1 is downloading from PROJECT1 in Terminal1 and immediately uploading into PROJECT2 in Terminal2.

In this way you dont have to wait that the full image is written to disk before starting the upload.

As you can now see the IMAGE1 is present in PROJECT2 glance service

```
(this command is issued on Terminal2 with PROJECT2 credentials)
# glance image-list 
+---------+-------------+-------------+------------------+--------------+--------+
| ID      | Name        | Disk Format | Container Format | Size         | Status |
+---------+-------------+-------------+------------------+--------------+--------+
| abc     | kali        | raw         | bare             | 2147483648   | active |
| def     | ubuntu 12   | raw         | bare             | 2147483648   | active |
| ghi     | Red Hat     | raw         | bare             | 2147483648   | active |
| 123     | IMAGE1      | raw         | bare             | 2147483648   | active |
+---------+-------------+-------------+------------------+--------------+--------+
````










































<img src="https://api.keen.io/3.0/projects/54bae2f990e4bd31acbe257d/events/trackme_airpair?api_key=8f78c2ac53f8fb11725b1262d0af9f339334d67581241b3d86469524b8cfc038091f8b1f9b2fd84f7e1a551900e87cc1f72cb9f835778aed02b3246b2f38a1f610fab167fda89e7cf3f906eac95dd4285135e72194e9e16e2bef7f40227c4ba98683419496c294396aeec368021a354e&data=ewogICAgImtlZW4iIDogewogICAgICAgICJhZGRvbnMiIDogWwogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAibmFtZSIgOiAia2VlbjppcF90b19nZW8iLAogICAgICAgICAgICAgICAgImlucHV0IiA6IHsKICAgICAgICAgICAgICAgICAgICAiaXAiIDogImlwX2FkZHJlc3MiCiAgICAgICAgICAgICAgICB9LAogICAgICAgICAgICAgICAgIm91dHB1dCIgOiAiaXBfZ2VvX2luZm8iCiAgICAgICAgICAgIH0KICAgICAgICBdCiAgICB9LAogICAgImlwX2FkZHJlc3MiIDogIiR7a2Vlbi5pcH0iLAogICAgInVzZXJfYWdlbnQiIDogIiR7a2Vlbi51c2VyX2FnZW50fSIsCiAgICAiaGl0IjogMSwKICAgICJwb3N0X2lkIjogMSwKICAgICJwb3N0IjogIk1vdmluZyBJbWFnZXMgb24gT3BlbiBTdGFjayIKfQo="></img>

