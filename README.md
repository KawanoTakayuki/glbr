# [Grouping Logs by Request](https://godoc.org/cloud.google.com/go/logging#hdr-Grouping_Logs_by_Request)

Google Cloud Platform: Stackdriver Logging client wrapper

## How to use

* simple logging

    ```golang
    package main

    import (
        "github.com/KawanoTakayuki/glbr"
    )

    func main() {
        log, err := glbr.NewLogging(context.Background(), "ProjectID", "LogID")
        if err != nil {
            panic(err.Error())
        }
        defer log.Close()

        glbr.Debugf(log.Context(), "log")
    }
    ```

* request grouping log

    ```golang
    package main

    import (
        "context"
        "net/http"

        "github.com/KawanoTakayuki/glbr"
    )

    func main() {
        logService, err := glbr.NewLogging(context.Background(), "ProjectID", "LogID")
        if err != nil {
            panic(err.Error())
        }
        defer logService.Close() // close only once

        /*
            each request
        */
        groupLog := logService.GroupedBy("ParentLogID")

        // group
        http.Handle("/group", groupLog(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            c := r.Context() // request context
            glbr.Debugf(c, "group log")
        })))

        // no group
        http.Handle("/no-group", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            c := logService.Context() // glbr context
            glbr.Debugf(c, "no group log")
        }))

        http.ListenAndServe(":8080", nil)

        /*
            all request
        */
        http.HandleFunc("/all-group", func(w http.ResponseWriter, r *http.Request) {
            c := r.Context() // request context
            glbr.Debugf(c, "all group log")
        })

        http.ListenAndServe(":8080", groupLog(http.DefaultServeMux))
    }
    ```

### [Monitored Resource Types](https://cloud.google.com/monitoring/api/resources)

* sample

    ```golang
    func main() {
        log, err := glbr.NewLogging(context.Background(), "ProjectID", "LogID")
        if err != nil {
            panic(err.Error())
        }
        defer log.Close()

        ResourceLavel := map[string]string{"key": "value"}
        log = log.Option(glbr.MonitoredResource("ResorceType", ResourceLavel))
        glbr.Debugf(log.Context(), "log")
    }
    ```
