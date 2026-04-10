GIT repo:

- [https://gitlab.zalopay.vn/aqr/bill](https://gitlab.zalopay.vn/aqr/bill)
- [https://gitlab.zalopay.vn/aqr/telco](https://gitlab.zalopay.vn/aqr/telco)

Jenkin: [https://jenkins.zalopay.vn/job/aqr](https://jenkins.zalopay.vn/job/aqr)

Docs: [https://confluence.zalopay.vn/x/0SSPAw](https://confluence.zalopay.vn/x/0SSPAw)

Services: [https://confluence.zalopay.vn/x/AzPVAw](https://confluence.zalopay.vn/x/AzPVAw)

  

# Build local

docker login [registry-gitlab.zalopay.vn](http://registry-gitlab.zalopay.vn/)

user: domain

pass: ssl token

# Tool

- Nacos
    
    ### Host
    
    10.40.81.10 [dev-nacos.zalopay.vn](http://dev-nacos.zalopay.vn/) [qc-nacos.zalopay.vn](http://qc-nacos.zalopay.vn/) [lt-nacos.zalopay.vn](http://lt-nacos.zalopay.vn/) [mc-nacos.zalopay.vn](http://mc-nacos.zalopay.vn/)
    
    ### dev: [https://dev-nacos.zalopay.vn/nacos/#](https://dev-nacos.zalopay.vn/nacos/#)
    
    nacos-mep / Nacos@2021
    
    ### MC: [https://mc-nacos.zalopay.vn/nacos/#](https://mc-nacos.zalopay.vn/nacos/#/login)
    
    nacos-mep / Nacos@2021
    
      
    
- CMDB
    
    Guide : [https://confluence.zalopay.vn/x/oHePAw](https://confluence.zalopay.vn/x/oHePAw)
    
    Tự escape kí tự (")
    
    ### Host
    
    10.30.94.60 [cmdb.zalopay.vn](http://cmdb.zalopay.vn/)
    
    ### Domain
    
    Domain [cmdb.zalopay.vn](http://cmdb.zalopay.vn/)
    
    Account: LDAP
    
      
    
      
    
- Logging
    - dev/qc/mc: rancher [https://zlp-dev-rancher.zalopay.vn/](https://zlp-dev-rancher.zalopay.vn/)
    - stg/real: [https://log.zalopay.vn](https://log.zalopay.vn/)
- Monitor
    
    [https://confluence.zalopay.vn/x/Nr1BAg](https://confluence.zalopay.vn/x/Nr1BAg)
    
    ### Host
    
    10.30.94.60 [grafana-backend.zalopay.vn](http://grafana-backend.zalopay.vn) [dashboard.zalopay.vn](http://dashboard.zalopay.vn/)
    
    ### Account
    
    dev/qc/mc: https://grafana-backend.zalopay.vn: `dev-pmet`/ `123@abc1`
    
    stg/real: https://dashboard.zalopay.vn: OTP
    
- tracing
    
    dev-jg.zpapps.vn
    
    dev / MEP@2018devtoqqazols@azxz!
    
    dev-kb.zpapps.vn
    
    dev / MEP@2018devtoqqazols@azxz!
    
    ```Bash
    golang project:
    
    import 	"go.opentelemetry.io/contrib/propagators/b3"
    
    func doStub(){
    	p := b3.New()
    	// Register the B3 propagator globally.
    	otel.SetTextMapPropagator(p)
    }
    
    
    // why wrong
    		propagator := otel.GetTextMapPropagator()
    ```
    
    ```Java
    TextMapGetter<HttpHeaders> getter =
      new TextMapGetter<HttpHeaders>() {
        @Override
        public String get(HttpHeaders headers, String s) {
          assert headers != null;
          return headers.getHeaderString(s);
        }
    
        @Override
        public Iterable<String> keys(HttpHeaders headers) {
          List<String> keys = new ArrayList<>();
          MultivaluedMap<String, String> requestHeaders = headers.getRequestHeaders();
          requestHeaders.forEach((k, v) ->{
            keys.add(k);
          });
        }
    };
    
    TextMapSetter<HttpURLConnection> setter =
      new TextMapSetter<HttpURLConnection>() {
        @Override
        public void set(HttpURLConnection carrier, String key, String value) {
            // Insert the context as Header
            carrier.setRequestProperty(key, value);
        }
    };
    
    //...
    public void handle(<Library Specific Annotation> HttpHeaders headers){
            Context extractedContext = opentelemetry.getPropagators().getTextMapPropagator()
                    .extract(Context.current(), headers, getter);
            try (Scope scope = extractedContext.makeCurrent()) {
                // Automatically use the extracted SpanContext as parent.
                Span serverSpan = tracer.spanBuilder("GET /resource")
                    .setSpanKind(SpanKind.SERVER)
                    .startSpan();
    
                try(Scope ignored = serverSpan.makeCurrent()) {
                    // Add the attributes defined in the Semantic Conventions
                    serverSpan.setAttribute(SemanticAttributes.HTTP_METHOD, "GET");
                    serverSpan.setAttribute(SemanticAttributes.HTTP_SCHEME, "http");
                    serverSpan.setAttribute(SemanticAttributes.HTTP_HOST, "localhost:8080");
                    serverSpan.setAttribute(SemanticAttributes.HTTP_TARGET, "/resource");
                    
                    HttpURLConnection transportLayer = (HttpURLConnection) url.openConnection();
                    // Inject the request with the *current*  Context, which contains our current Span.
                    openTelemetry.getPropagators().getTextMapPropagator().inject(Context.current(), transportLayer, setter);
                    // Make outgoing call
                }finally {
                    serverSpan.end();
                }
          }
    }
    ```
    

# ENV

Kafka: 10.109.3.52:9092,10.109.3.53:9092,10.109.3.54:9092

  

# DB convention

[telco/bill]_[service]_dev

[telco/bill]_[service]_qc

[telco/bill]_[service]_mc

[telco/bill]_[service]_stg

[telco/bill]_[service]_real

ex: telco_postpaid_provider_stg

# Commute

call to service chung cụm

service_name.namespace.svc:port

[http://zp-cps-provider-payoo.aqr-bill-dev.svc:80](http://zp-cps-provider-payoo.aqr-bill-dev.svc/)

  

git-projectname.namesapace.svc = provider-hue.aqr-bill-dev.svc

servervice_name = git_project

  

# Coverage

I had generated the attached file (coverage.out) by command

```Plain
go test -v -coverpkg=./... $(go list ./... | grep -v /test ) -coverprofile=coverage.out -covermode=count -json
```

you can follow this link [https://confluence.zalopay.vn/x/gKFHAw](https://confluence.zalopay.vn/x/gKFHAw) for more detail.

And for checking, just execute this command:

```Plain
go tool cover -html=coverage.out
```

# Tags

format: **v**x.y.z

ex: v1.0.0

  

  

## Grafana

```Bash
1, add host

10.30.94.60 grafana-backend.zalopay.vn

 
2, https://grafana-backend.zalopay.vn/

        dev-pmet    /   123@abc1
```

  

#### Progress

|Name|Tags|Type|
|---|---|---|
|[[PHU-HOA-TAN]]|Processing|Water|
|[[Provider ThuDuc]]|Processing|Water|
|[[NewENV CheckList]]|Processing|Electricity|
|[[provider-shinhan]]|dev||
|[[provider-evnnpc]]|dev, prod, stg||
|[[provider-evnspc]]|dev, prod||
|[[provider-namdinh]]|dev||
|[[provider-vinhlong]]|dev||
|[[CheckList]]|dev||
|[[Core-api]]|dev, stg||
|[[Provider Payoo]]|dev, stg||
|[[Provider VNPAY]]|dev||