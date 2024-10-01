apiVersion: batch/v1
kind: CronJob
metadata:
  name: exportreaper
  labels:
    app: exportreaper
spec:
  schedule: "0/1 * * * *" # Example schedule: daily at 17:00 UTC
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: exportreaper
        spec:
          containers:
            - name: exportreaper
              image: "{{ .Values.exportreaper.image.repository }}:{{ .Values.exportreaper.image.tag }}"
              imagePullPolicy: {{ .Values.exportreaper.image.pullPolicy }}
              command: ["/bin/sh", "-c", "/root/exporter/run.sh"]
              volumeMounts:
                - mountPath: /usr/local/share/ca-certificates
                  name: bundle
              resources:
                requests:
                  cpu: {{ .Values.exportreaper.resourceLimits.requests.cpu }}
                  memory: {{ .Values.exportreaper.resourceLimits.requests.memory }}
                limits:
                  cpu: {{ .Values.exportreaper.resourceLimits.limits.cpu }}
                  memory: {{ .Values.exportreaper.resourceLimits.limits.memory }}
              args: [
                "--zk_hosts",
                "sigma.main.Main",
                "-c",
                "exporter.properties",
                "-s",
                "ExportReaper",
                "-Dexport.manager.host=export.csn.svc.cluster.local"

              ]
              env:
                - name: LOG_DRIVER
                  value: awslogs 
                - name: LOG_GROUP
                  value: /sigma/mesos 
                - name: LOG_STREAM
                  value: exportreaper
                - name: AUTH.API.TOKEN
                  valueFrom:
                    secretKeyRef:
                      name: sigmasecret
                      key: EXPORTER_API_TOKEN
                - name: PROFILE_CACHE_NULL_TIMEOUT
                  value: "1000"
                - name: NOTIFICATIONS_API_URL
                  value: "http://notifications.csn.svc.cluster.local:4343"
                - name: PROFILE_SERVICE_URL
                  value: "http://profile-manager.csn.svc.cluster.local:8012/sigma/profile/v1"                  
                  
                  
          volumes:
            - name: bundle
              configMap:
                name: {{ .Values.exportreaper.bundle.name }}
          restartPolicy: OnFailure
