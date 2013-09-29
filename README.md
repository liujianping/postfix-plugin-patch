Postfix DICT Plugin Howto

based on postfix 2.7.2
-------------------------------------------------------------------------------

Introduction

The Postfix plugin map type allows you to extend dict map type as you like,
and do not patch the postfix source code any more, just implement the plugin interface.

Building Postfix with Http support

Get postfix source code from:
    http://www.postfix.org/download.html

    tar xfz postfix-2.x.x.tar.gz
    cd postfix-2.x.x
    patch -p1 -d ./ < ../postfix-2.x.x_with_dict_plugin.patch

Then you can build Postfix from source code as described in the INSTALL document. 

Using Plugin map

Once Postfix is built with http support, you can specify a map type in main.cf
like this:

    virtual_mailbox_maps = plugin:/etc/postfix/plugin_mailbox.cf

The file /etc/postfix/mailbox.cf specifies lots of information telling
Postfix how to configure the plugin.

Plugin Configure Example:

		@/etc/postfix/plugin_mailbox.cf
		-------------------------------------------------------------------
		1 plugin_name = /etc/postfix/plugins/libmailbox.so
		2 open = name_of_init_fuction_in_plugin
		3 open_arg = param_of_init_fuction_in_plugin
		4 close = name_of_final_fuction_in_plugin
		5 lookup = name_of_lookup_fuction_in_plugin
		6 update = name_of_update_fuction_in_plugin
		7 delete = name_of_delete_fuction_in_plugin
		8 sequence = name_of_sequence_fuction_in_plugin

line 1: tell the postfix where to dlopen the plugin, 
			 	of course you can just set the plugin name without absolute path, 
			 	you should make sure the plugin in the ld.conf path where dlopen can find

line 2: name of the plugin to initialize some resource, the item is option
line 3: argument string of the plugin to initialize fuction, the item is option
line 4: name of the plugin to finalize the resource we initialized, the item should be with the initialize option,
				otherwise there may be memory leak problem.

line 5:	name of the plugin fuction to lookup a key 
line 6:	name of the plugin fuction to update a key 
line 7:	name of the plugin fuction to delete a key 
line 8:	name of the plugin fuction to sequence <key, value> pair 

if line 5 - 8, some or all items not defined, plugin will use the default dict method(which implemention is not support).
and line 2 - 3, all are optional. But you should be attention, when you create a resource, you should release it.

Plugin Map Program Guide:

1. phototypes of the plugin interface
	 1) open phototype:
	 		definition:
	 								void* (*open)(const char* arg);
	 		description:
						@Usage: create a resource obj of the plugin, will used by the other fuctions.
						@Param: init_param, we can define something useful when create the resource. 
										and the init_param will be readed from the plugin configure file, reference to [Plugin Configure Example: line 3]										 							
	 					@Result: return the resoure object ptr, 
	 									 Null, means create the resource object failed
	 									 Not Null, means ok.
	 					
	 2) close phototype:
	 		definition:
	 								void (*close)(void* res);
	 		description:
						@Usage: destroy a resource obj
						@Param: res, created by open fuction.

	 3) lookup phototype:
	 		definition:
	 								const char* (*lookup)(void* res, const char* key);
	 		description:
						@Usage: lookup the value of the dest key
						@Param: res, created by open fuction.
										key, postfix provide the param when invoke the fuction
						@Result:
										value of the type const char*;
										Null, means not find.
										Not Null, finded.
										
	 4) update phototype:
	 		definition:
	 								int (*update)(void* res, const char* key, const char* value);
	 		description:
						@Usage: update the value of the dest key with the dest value
						@Param: res, created by open fuction.
										key, postfix provide the dest key when invoke the fuction
										value, postfix provide the dest value when invoke the fuction
						@Result:
										-1, fuction failed
										0, fuction succeed with no key update
										1, fuction succeed with one key update
	 		
	 5) delete phototype:
	 		definition:
	 								void* (*delete)(void* res, const char* key);
	 		description:
						@Usage: delete the the dest key pair 
						@Param: res, created by open fuction.
										key, postfix provide the dest key when invoke the fuction
						@Result:
										-1, fuction failed
										0, fuction succeed with no key deleted
										1, fuction succeed with one key deleted

	 6) sequence phototype:
	 		definition:
	 								void* (*sequence)(void* res, int i, const char ** key, const char **value);
	 		description:
	 					this interface just for the dict sequence interface, many inherit dict map doesnt support the function.
	 					if you want to implement the interface, just make sure you know more detail enough. 
	 					you can reference to the postfix/src/util/dict.h file.
	 			
	 
2. implemetion your plugin.
	 :)
		
Credits

    JianPing Liu
    liujianping.china@gmail.com  
    Shanghai, PRC 
    
    This project: 
