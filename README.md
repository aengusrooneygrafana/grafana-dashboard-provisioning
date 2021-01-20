Provisioning Grafana dashboards using the REST API 

** TEST ONLY - TESTED ON DOKER ONLY ** 

Note:  The identifier (id) of a dashboard is an auto-incrementing numeric value and is only unique per Grafana install.  The unique identifier (uid) of a dashboard can be used for uniquely identify a dashboard between multiple Grafana installs.  It’s automatically generated if not provided when creating a dashboard.  The uid allows having consistent URLs for accessing dashboards and when syncing dashboards between multiple Grafana installs, see dashboard provisioning for more information.  This means that changing the title of a dashboard will not break any bookmarked links to that dashboard.  Ref:  https://grafana.com/docs/grafana/latest/http_api/dashboard/ 

1.  Get a list of all dashboard id's from Grafana host 

```curl -X GET -H "Authorization: Bearer <token>" -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:3000/api/search?query=% | jq '.[] | .uid' > dashboards-uid.lst ```

2.  Remove quotes from the UID's  

```sed -i '' 's/\"//g' dashboards-uid.lst ```

3.  Get the json for that list of dashboard UID's 

```for i in `cat dashboards-uid.lst` ; do curl -X GET -H "Authorization: Bearer <token>" -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:3000/api/dashboards/uid/$i > dashboard.$i.json ; done ```

4.  Delete existing dashboards (only if testing against the same host.  If using a second host, this is not necessary).  

```for i in `cat dashboards-uid.lst` ; do curl -X DELETE -H "Authorization: Bearer <token>" -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:3000/api/dashboards/uid/$i ; done ```

5.  Create dashboards on Grafana host  

```
for i in *.json; do cat $i | jq '.dashboard.id = null' > create-update.$i ;  
curl -X POST -H "Authorization: Bearer <token>" -H "Accept: application/json" -H "Content-Type: application/json" -d @create-update.$i http://localhost:3000/api/dashboards/db ; 
done 
``` 
