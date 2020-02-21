# Readme IOTA
This readme is a guide to using the scripts that mass provision the Kong service in a POD, which follows the following process:

   * List the IDs of the services currently configured in the pod
   * List the IDs of the current routes configured in the pod
   * Generate commands to remove services
   * Generate commands to eliminate routes
   * Generate the commands to create the routes again
   
Each of these steps is done with a different script, which are detailed below:

## List the IDs of current services
It is necessary to list all the IDs of the current services to later remove them, for this the list_kong_services script is used, which is a Shell script. This script does not need to receive parameters in its execution. Which lists the installed kongs and takes the first. Then execute the curl command to get all the services and finally filter only the IDs of each service and display them in the console.

## List the IDs of the current routes
It is necessary to list all the IDs of the current routes to later eliminate them, for this the list_kong_routes script is used which is a Shell script. This script does not need to receive parameters in its execution. Which lists the installed kongs and takes the first. Then execute the curl command to get all the services and finally filter only the IDs of each route and display them in the console.

## Generate commands to remove services
After generating the service IDs, the Shell gen_delete_kong_services script, which receives the information that was previously generated with the list_kong_services script as a parameter for execution. This script locates and selects the first installed Kong. Then use the curl -k -X DELETE command, with which the command is generated to remove each of the previously listed services.

## Generate commands to remove routes
After generating the IDs of the routes, the Shell script gen_delete_kong_routes, which receives as a parameter for the execution the information that was previously generated with the script list_kong_routes. This script locates and selects the first installed Kong. Then use the curl -k -X DELETE command, with which the command is generated to remove each of the previously listed routes.

Generate the commands to create the routes again
Finally, the new routes must be generated, for this the awk script, gen_create_kong_services.awk is used, which receives the following parameters, where the order is important:
1. cvs file, converted from the mapping.xlsx file
2. pod_name
3. api = Kong’s admin url, default “http: // localhost: 8001”
4. vip = Destination ip or name
5. port = Destination port
6. version = Destination version in url


"The script"
The script extracts from the cvs file the information required to generate the commands to create the new routes, as follows:
• Validate the name of the section
• Each line makes it lowercase
• Remove any character other than letter, number or script from each line
• Eliminate the spaces of each line
• Identify whether it is internal or external
• Insert the new external domain
• Identify the Rest method of each line
• Print a line on the screen to identify the following command
• With all the information previously extracted, the command to create each route is generated
“Command example”
