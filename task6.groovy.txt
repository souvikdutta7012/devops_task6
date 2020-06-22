job("job1") {


  description("Creates the image")
  
  steps {
  
   scm {
     git {
       extensions {
         wipeWorkspace()
       }
     }
   }
   scm {
     github("souvikdutta7012/devops_task6", "master")
   }
   shell("sudo cp -rvf * /root/devopstask6;if ls | grep php;then sudo cp /root/devopstask6/image/php/Dockerfile /root/devopstask6/;else sudo cp /root/devopstask6/image/html/Dockerfile /root/devopstask6/;fi")
   shell("sudo docker build -t souvikdutta7012/website:v1 /root/devopstask6/;sudo docker push souvikdutta7012/website:v1")  
  }
}



job("job2") {


  description("Deploying job")
    
  triggers {
    upstream {
      upstreamProjects("job1")
      threshold("SUCCESS")
    }  
  }

  steps {
      shell("if sudo kubectl get pods | grep mypod; then sudo kubectl delete pods --all; fi;if sudo kubectl get service | grep myservice ; then sudo kubectl delete service --all; fi;if sudo kubectl get pvc | grep mywebpvc; then sudo kubectl delete pvc --all; fi;kubectl create -f /root.devopstask6/mypvc.yml;kubectl create -f /root/devopstask6/deploy.yml; kubectl create -f /root/devopstask6/service.yml;kubectl get all;") 

  }
}




job("job3") {
  
  description("Monitoring Job")


  triggers {
     scm("* * * * *")
   }


  steps {
    
    shell('export status=$(curl -siw "%{http_code}" -o /dev/null 192.168.99.100:30000); if [ $status -eq 200 ]; then exit 0; else python3 sendmail.py; exit 1; fi')

  }
}





job("job4") {


  description("Redeploying Job")


  triggers {
    upstream {
      upstreamProjects("job3")
      threshold("FAILURE")
    }
  }
  
  
  publishers {
    postBuildScripts {
      steps {
        downstreamParameterized {
  	  	  trigger("job1")
        }
      }
    }
  }
}