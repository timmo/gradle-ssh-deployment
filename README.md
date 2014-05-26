gradle-ssh-deployment
=====================

Deployment via "rolling update" implemented with gradle ssh plugin.

See also http://gradle-ssh-plugin.github.io

```
remotes {
  Closure id_rsa = {
    file("${System.properties['user.home']}/.ssh/id_rsa")
  }
  at1 {
    role 'at' // automated test stage
    host = 'myservice1.at.local'
    user = 'myservice'
    identity = id_rsa.call()
  }
  at2 {
    role 'at'
    host = 'myservice2.at.local
    ...
  }
  ps1 { ... } // production stage
  ps2 { ... }
}

['at', 'ps'].each { stage ->
  task "deploy${stage}"(type: SshTask) {
    String artefaktUri = ...  // points to maven repository
    session(remotes.role(stage)) {
      execute('if [ -x service.sh ]; then ./service.sh stop; fi')
      put('pipeline/deployable/downloadAndUnpack.sh', './')
      execute("chmod 755 ./downloadAndUnpack.sh; ./downloadAndUnpack.sh ${artefaktUri}")
      execute("./service.sh start", pty: true)
      execute("./waitForServiceStart.sh")
    }
  }
}

```
