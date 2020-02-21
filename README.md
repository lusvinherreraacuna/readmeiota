# Readme IOTA
This readme is a guide to using the scripts that mass provision the Kong service in a POD, which follows the following process:

   * List the IDs of the services currently configured in the pod
   * List the IDs of the current routes configured in the pod
   * Generate commands to remove services
   * Generate commands to eliminate routes
   * Generate the commands to create the routes again
   
Each of these steps is done with a different script, which are detailed below:

## List the IDs of current services
It is necessary to list all the IDs of the current services to later remove them, for this the **list_kong_services** script is used, which is a Shell script. This script does not need to receive parameters in its execution. Which lists the installed kongs and takes the first. Then execute the curl command to get all the services and finally filter only the IDs of each service and display them in the console.

### list_kong_services
```
#       List kong services's ids
#       SESS - 2020-02-18

# TODO: how to use functions declared in session?
KUBE_FILE=`ls | grep kubeconfig`
k8s ()
{
    kubectl --kubeconfig ~/$KUBE_FILE -n iotaccelerator "$@"
}

pod=`k8s get pods | grep kong-rc | head -1 | awk '{ print $1 }'`
next=/services
all_services=""
while [ "$next" != "" ]
do
  services=`k8s exec $pod -i -t -- curl -k --url http://localhost:8001$next`
  list=`echo $services | jq '.data[].id'`
  if [ "$all_services" == "" ]; then
    all_services=$list
  else
    all_services="$all_services $list"
  fi
  next=`echo $services | jq '.next'`
  if [ "$next" == "null" ]; then
    next=
  else
    next=`echo $next | tr -d '"'`
  fi
done
echo $all_services
```
_Command example_
```
“Command example”
```


## List the IDs of the current routes
It is necessary to list all the IDs of the current routes to later eliminate them, for this the **list_kong_routes** script is used which is a Shell script. This script does not need to receive parameters in its execution. Which lists the installed kongs and takes the first. Then execute the curl command to get all the services and finally filter only the IDs of each route and display them in the console.

### list_kong_routes
```
#       List kong routes's ids
#       SESS - 2020-02-18

# TODO: how to use functions declared in session?
KUBE_FILE=`ls | grep kubeconfig`
k8s ()
{
    kubectl --kubeconfig ~/$KUBE_FILE -n iotaccelerator "$@"
}

pod=`k8s get pods | grep kong-rc | head -1 | awk '{ print $1 }'`
next=/routes
all_routers=""
while [ "$next" != "" ]
do
  routes=`k8s exec $pod -i -t -- curl -k --url http://localhost:8001$next | jq '.'`
  list=`echo $routes | jq '.data[].id'`
  if [ "$all_routes" == "" ]; then
    all_routes=$list
  else
    all_routes="$all_routes $list"
  fi
  next=`echo $routes | jq '.next'`
  if [ "$next" == "null" ]; then
    next=
  else
    next=`echo $next | tr -d '"'`
  fi
done
echo $all_routes
```
_Command example_
```
“Command example”
```


## Generate commands to remove services
After generating the service IDs, the Shell **gen_delete_kong_services** script, which receives the information that was previously generated with the list_kong_services script as a parameter for execution. This script locates and selects the first installed Kong. Then use the curl -k -X DELETE command, with which the command is generated to remove each of the previously listed services.

### gen_delete_kong_services
```
#       Generate commands fro delete kong's services
#       SESS - 2020-02-19

# TODO: how to use functions declared in session?
KUBE_FILE=`ls | grep kubeconfig`
k8s ()
{
    kubectl --kubeconfig ~/$KUBE_FILE -n iotaccelerator "$@"
}

pod=`k8s get pods | grep kong-rc | head -1 | awk '{ print $1 }'`
all_services=`./list_kong_services`

for service in $all_services
do
  service=`echo $service | tr -d '"'`
  echo "k8s exec $pod -i -t -- curl -k -X DELETE http://localhost:8001/services/$service"
done
```
_Command example_
```
“Command example”
```

## Generate commands to remove routes
After generating the IDs of the routes, the Shell script **gen_delete_kong_routes**, which receives as a parameter for the execution the information that was previously generated with the script list_kong_routes. This script locates and selects the first installed Kong. Then use the curl -k -X DELETE command, with which the command is generated to remove each of the previously listed routes.

### gen_delete_kong_routes
```
#       Generate commands fro delete kong's routes
#       SESS - 2020-02-19

# TODO: how to use functions declared in session?
KUBE_FILE=`ls | grep kubeconfig`
k8s ()
{
    kubectl --kubeconfig ~/$KUBE_FILE -n iotaccelerator "$@"
}

pod=`k8s get pods | grep kong-rc | head -1 | awk '{ print $1 }'`
all_routes=`./list_kong_routes`

for route in $all_routes
do
  route=`echo $route | tr -d '"'`
  echo "k8s exec $pod -i -t -- curl -k -X DELETE http://localhost:8001/routes/$route"
done
```
_Command example_
```
“Command example”
```


## Generate the commands to create the routes again
Finally, the new routes must be generated, for this the awk script, **gen_create_kong_services.awk** is used, which receives the following parameters, where the order is important:
1. cvs file, converted from the mapping.xlsx file
2. pod_name
3. api = Kong’s admin url, default “http: // localhost: 8001”
4. vip = Destination ip or name
5. port = Destination port
6. version = Destination version in url

### gen_create_kong_services.awk
```

#       Generate create services and routes for kong
#       SESS - 2020-02-19

# Variables from command line:
#       - pod_name
#       - api           Kong's admin url, default 'http://localhost:8001'
#       - vip           Destination ip or name
#       - port          Destination port
#       - version       Destination version in url
#
# Remeber to set FS, depending on regional settings Excel may separate fields using semicolon instead of comma.

BEGIN {
  FS = ",";
  if (api == "") {
    api = "http://localhost:8001"
  }
  vip_and_port = vip ":" port
  if (version == "") {
    version = "1"
  }
}
{
  # Mappings are numbered
  if ($1 ~ /[0-9]+/) {
    name = tolower(sec) "-" tolower($2) "-" version "-" $1
    name = gensub(/[^a-zA-Z0-9\- ]/, "", "g", name)
    name = gensub(/ /, "-", "g", name)
    external = gensub(/<version>/, version, "g", $3)
    internal = gensub(/<vip:port>/, vip_and_port, "g", $4)
    # do we have destination?
    if (internal != "") {
      internal = gensub(/<version>/, version, "g", internal)
      external = gensub(/https:\/\/<external_domain>/, "", "g", external)
      get = 0; post = 0; put = 0; delete_ = 0; have_methods = 0;
      method = $5
      if (method != "") {
        get = index(method, "GET")
        post = index(method, "POST")
        put = index(method, "PUT")
        delete_ = index(method, "DELETE")
        have_methods = get + post + put + delete_
      }
      print "# mapping", name, external, ">", internal, method
      # Service:
      print "k8s exec " pod " -- curl -H 'Content-Type: application/json' -X POST " api \
        "/services --data '{ \"name\": \"" name "\", \"url\": \"" internal "\" }'"
      # Route:
      methods = ""
      if (have_methods > 0) {
        methods = ", \"methods\": ["
        if (get > 0) {
          methods = methods "\"GET\","
        }
        if (post > 0) {
          methods = methods "\"POST\","
        }
        if (put > 0) {
          methods = methods "\"PUT\","
        }
        if (delete_ > 0) {
          methods = methods "\"DELETE\","
        }
        methods = gensub(/,$/, "", "g", methods) "]"
        print "k8s exec " pod " -- curl -H 'Content-Type: application/json' -X POST " api \
          "/services/" name "/routes --data '{ \"paths\": \"" external "\"" methods " }'"
      }
    }
  } else {
    # is section name
    sec = $1
  }
}

```
The script extracts from the cvs file the information required to generate the commands to create the new routes, as follows:
* Validate the name of the section
* Each line makes it lowercase
* Remove any character other than letter, number or script from each line
* Eliminate the spaces of each line
* Identify whether it is internal or external
* Insert the new external domain
* Identify the Rest method of each line
* Print a line on the screen to identify the following command
* With all the information previously extracted, the command to create each route is generated

_Command example_
```
“Command example”
```
