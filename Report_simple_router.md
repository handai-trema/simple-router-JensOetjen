# Report: Simple Router #

Date: 14.11.2016 <br />
Name: Jens Oetjen <br />
Student Number: 33E16024 <br />

## 1 Introduction ##

In this assignment we were supposed to create write a comannd line interface for a router. The router was supposed to have the following
minimal functions.

1. Show the routing table on the router

2. Add/delete entries to/from the routing table

3. Show a list of all interfaces on the router

As an additional function I implemented that the router can made a backup of its routes in a file and recover the routes from this backup. 

## 2 Implementation of the solution ##

The follwing code shows the code that was used for showing of the routing tables. The CLI-function calls the function return table() which calls
@routing table.getDB() to access the array with the routes. The resulting area is splitted, formed into strings and send as output:

Commando-function

```ruby
desc 'show_entries'
  arg_name ''
  command :show_entries do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|
    
      
       @entries= Trema.trema_process('SimpleRouter', options[:socket_dir]).controller. return_table()
       #pp @entries
      
      print "Destination \t \t Next Hop \n"
          
      print "-------------------------------------------------------- \n"
      
      @entries.each do |each|
        each.each_key do |key|
          print IPv4Address.new(key).to_s, "/", @entries.index(each), "\t  \t", each[key].to_s, "\n"
        end
        
      end
      
    end
 end  
 ```
Source-function

```ruby
def return_table()
    return @routing_table.getDB()
  end
  ```
  
 New routes can be added via command line. There are three parameters that need to be given. These are destination network, netmask and address of the next hop:
 
 Commando-function
 
 ```ruby
desc 'add entry'
  arg_name 'dest_ip nmask ip_n_hop'
  command :add_entry do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|      
      
      dest_ip = args[0]
      nmask = args[1]
      ip_n_hop = args[2]
    
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller. add_entry(dest_ip,nmask,ip_n_hop)
        
    end
 end 
 ```
Source-function

```ruby
def add_entry(dest_ip, nmask, ip_n_hop)
    parameters = {:destination => dest_ip, :netmask_length => nmask.to_i, :next_hop => ip_n_hop}
    @routing_table.add(parameters)
  end
  ```
  
 Routes can be deleted via command line. The commando requires 2 arguments. These are destination network and netmask. Another function that does the actual
 deleting needed to be added to routing table rb.
 
 Commando-function
 
 ```ruby
   desc 'delete entry'
  arg_name 'dest_ip nmask'
  command :delete_entry do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|      
      
      dest_ip = args[0]
      nmask = args[1]      
    
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller. delete_entry(dest_ip,nmask)        
    end
 end  
 ```
Source-function

```ruby
 def delete_entry(dest_ip, nmask)
    parameters = {:destination => dest_ip, :netmask_length => nmask.to_i}
    @routing_table.delete(parameters)
  end
  ```
  
  Source-function in routing table.rb

```ruby
def delete(options)
    netmask_length = options.fetch(:netmask_length)
    prefix = IPv4Address.new(options.fetch(:destination)).mask(netmask_length)
    @db[netmask_length].delete(prefix.to_i)
  end
  ```
  
I implemented the function to write all routing entries into a file called backup.txt. Each enrty is written in a single line, the elements are seperated by whitespace.
Whenever the function is called, the old file is overwritten:
 
 Commando-function
 
 ```ruby
   desc 'write route'
  arg_name ''
  command :write_route do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller. write_route()        
    end
end  
 ```
Source-function

```ruby
def write_route()
      out_file = File.new("backup.txt", "w")
     
     @entries= return_table()
     
     @entries.each do |each|
        each.each_key do |key|
         out_file.print(IPv4Address.new(key).to_s, " ", @entries.index(each), " ", each[key].to_s," \n")
        end        
      end    
	out_file.close
    end 
  end
  ```
  
The routes can be imported easily with import route. The lines are splitted by whitespace and every route is added to the routing table.
 
 Commando-function
 
 ```ruby
   desc 'import route'
  arg_name ''
  command :import_route do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args| 
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller. import_route()
    end
end 
 ```
Source-function

```ruby
def import_route()  
    file = File.new("backup.txt", "r")
    while (line = file.gets)
     values=line.split(" ")
     add_entry(values[0],values[1],values[2])  
    end
  file.close
 end
  ```

I will show that my code works with the follwing commandos. First I will show that adding a route works. I will then back the routes and delete the new route. 
In the last step I will import the routes again.:

```	
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli show_entries
Destination 	 	 Next Hop 
-------------------------------------------------------- 
0.0.0.0/0	  	192.168.1.2
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli add_entry 192.168.5.0 24 192.168.1.1
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli show_entries
Destination 	 	 Next Hop 
-------------------------------------------------------- 
0.0.0.0/0	  	192.168.1.2
192.168.5.0/24	  	192.168.1.1
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli write_route
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli delete 192.168.5.0 24
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli show_entries
Destination 	 	 Next Hop 
-------------------------------------------------------- 
0.0.0.0/0	  	192.168.1.2
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli import_route
root@Jens-Oetjen-PC:/home/jens/Desktop/lesson5/simple_router# ./bin/router_cli show_entries
Destination 	 	 Next Hop 
-------------------------------------------------------- 
0.0.0.0/0	  	192.168.1.2
192.168.5.0/24	  	192.168.1.1
 ```
 
