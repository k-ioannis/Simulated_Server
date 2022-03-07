#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <pthread.h>
#include <sys/time.h>
#include<time.h>
#define MAX_LINE 25

//a simple struct containing the
//[request.id] . [request.size]
//information we'll need later
typedef struct t_Request
{
	int id;
	int size;
}t_Requests;

//A struct used to pass 
//many arguements
//inside a thread
typedef struct t_Arguement
{
	t_Requests * request;
	int cache;
	int * cache_array;
	long int * ttl_array;
	int id;
	int sum;
	int index;
	int flushed;
}t_Arguement;

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~|T  H  R  E  A  D  S|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//
//~~~~~~~~~~~~~~~~~~~~~~~~~|F  U  N  C  T  I  O  N  S|~~~~~~~~~~~~~~~~~~~~~~~~~~//
//All the file asosciated functions declared here 
//and are found after then driver main 
//just to keep them distinct from the thread asosciated ones
int get_Requests( FILE * probe );
t_Requests * convert_Req( FILE * probe, int count);
int get_Settings(int * thread_Setting, int * cache_Setting);


//Decleration of the mutual exclusion
//and the condition variable
static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

//A function to find if theres an occurence of a request
//inside the cache memory
int occurance(t_Arguement * arguement, int target, int limit)
{
	int i;
	for(i = 0; i < limit; i++)
	{
		if(target == arguement->cache_array[i] )
		{
			printf("\n[%d] REQUEST ALREADY IN QUEUE",  arguement->id);
			return -1;
		}
		else
		{
			printf("\n[%d] QUEUE DOESNT COTNAIN REQUEST\n    PROCEED...",  arguement->id);
			
			//sleep(arguement->request[i].size / 10);
			//colud not get to work so..
			sleep(1);
			return 0;	
		}
	}



}


//The fifth *in this case* thread 
//responsible to find the oldest request
//and then remove it from all the arrays

void * cleanup_Routine(void * arg)
{
	t_Arguement * argument = (t_Arguement *) arg; 
	printf("\nHELLO FROM CLEANUP");
	//printf("\n%d", argument->index);
	
	int i, max_index = 0; 
	long int diference = 0;
	clock_t t0 = time(0) * 1000;
	
	//A "simple" for loop to find the request 
	//witj the longest time stamp in the 
	for(i = 0; i < argument->index; i++)
	{
		//printf("\ntemp  = [%ld] - [%ld]",t0, argument->ttl_array[i]);
		long int  temp = (t0 - argument->ttl_array[i]);
		//printf("\nIS [%ld] > [%ld] ?", argument->ttl_array[i], temp);
		if(argument->ttl_array[i] > temp)
		{
			diference = temp;
			max_index = i;
		
		}	
		
	}
	
	//Prompting to show which request gow deleted
	printf("\nDELETING REQUEST %d", argument->request[max_index].id );
	
	//now removing it from all arrays
	//[argument->cache_array] . [argument->ttl_array] . [argument->requests] 
	//while restoring memory to the cahe
	//[argument->cache]
	argument->cache += argument->request[max_index].size;
	argument->request[max_index].id = -1;
	argument->ttl_array[max_index] = 0;
	argument->cache_array[max_index] = -1;
	
	//and updating a flag 
	//insdide the arguement struct
	//so that the other threads 
	//dont skip a request
	argument->flushed = 0;
}





void cache_insert(t_Arguement * arguement, int i)
{
	if(i == 0)
	{
		//
		printf("\n[%d] [PULLED REQUEST: %d]", arguement->id, arguement->request[i].id);
		printf("\n[%d] SEARCHING CACHE..", arguement->id);

		//Have to malloc space for the cache array 
		//[arguement->request...] 
		arguement->cache_array = (int *)malloc(sizeof(int));
		arguement->cache_array[0] = arguement->request[i].id;
		
		//same goes to ttl_array inside which the timestamps are stored
		//[arguement->ttl_array...]	
		arguement->ttl_array = (long int *)malloc(sizeof(long int));
		clock_t t0 = time(0);
		arguement->ttl_array[i] = t0 * 1000;
		printf("\n[%d] [TIMED ENTRY  %ld]", arguement->id, arguement->ttl_array[i]);
		
		//and finaly subtracting the size of each request from the cache memory 
		arguement->cache -= arguement->request[i].size;
		printf("\n[%d] [CACHE CAPACITY %d]", arguement->id, arguement->cache);			
		
		arguement->request[i].size = -1;
	}else
	{
	
		printf("\n[%d] [PULLED REQUEST: %d]", arguement->id, arguement->request[i].id);
		printf("\n[%d] SEARCHING CACHE", arguement->id);
		
		//First prioty has the occurance of the request in the cache queue
		//so a function is called to check the occurance
		int safeguard = occurance(arguement, arguement->request[i].id, i);
		
		if(safeguard == 0)
		{
			//NOW if there is memory still left then we keep adding up in the array 
			if((arguement->cache - arguement->request[i].size) > 0)
			{
				
				printf("\n[%d] MEMORY SUFICIENT", arguement->id);
				
				//Have to malloc space for the cache array 
				//[arguement->request...] 
				arguement->cache_array = (int *)malloc((i + 1) * sizeof(int));
				arguement->cache_array[i] = arguement->request[i].id;
				
				//same goes to ttl_array inside which the timestamps are stored
				//[arguement->ttl_array...]				
				arguement->ttl_array = (long int *)malloc((i + 1) * sizeof(long int));
				clock_t t0 = time(0);
				arguement->ttl_array[i] = t0 * 1000;
				printf("\n[%d] [TIMED ENTRY  %ld]", arguement->id, arguement->ttl_array[i]);
		
				arguement->cache -= arguement->request[i].size;
				printf("\n[%d] [CACHE CAPACITY %d]", arguement->id, arguement->cache);	
			
				arguement->request[i].size = -1;
			}else
			{
				//IF not then we must call cleanup to realse some 
				//so a new thread is created and its target is
				//this time is the cleanup routine 
				printf("\n[%d] INSUFICIENT MEMORY", arguement->id);
				arguement->index = i;
				pthread_t cleanup;
				pthread_create(&cleanup,NULL, cleanup_Routine, arguement);
				pthread_join(cleanup, NULL);
				

				printf("\n\n[%d] MEMORY SUFICIENT", arguement->id);
				
				//Have to malloc space for the cache array 
				//[arguement->request...] 
				arguement->cache_array = (int *)malloc((i + 1) * sizeof(int));
				arguement->cache_array[i] = arguement->request[i].id;
				
				//same goes to ttl_array inside which the timestamps are stored
				//[arguement->ttl_array...]				
				arguement->ttl_array = (long int *)malloc((i + 1) * sizeof(long int));
				clock_t t0 = time(0);
				arguement->ttl_array[i] = t0 * 1000;
				printf("\n[%d] [TIMED ENTRY  %ld]", arguement->id, arguement->ttl_array[i]);
		
				arguement->cache -= arguement->request[i].size;
				printf("\n[%d] [CACHE CAPACITY %d]", arguement->id, arguement->cache);	
			
				arguement->request[i].size = -1;
				
			}	
		}
	}
}

//
void * handle_Request(void * arg)
{
	//casting the [void *] arguement to the proper type
	//and then the code is as follows
	t_Arguement * arguement = (t_Arguement *) arg;
	
	
	int  i = 0;
	
	printf("\n\nHELLO FROM THREAD[%d]", arguement->id);
	
	//a loop to parse throigh all of the requests
	//found in [arguement->requests]
	
	while(1)
	{
		arguement->flushed = -1;
		if(i == arguement->sum)
			pthread_exit(0);

		//Each completed request gets branded by a -1 value
		//just to be safe they wont get parsed and completed
		//a 2nd time	
		if(arguement->request[i].size != -1 )
		{
			//Locking the mutex so the  
			//[cache_insert] function can execute safely
			//so one thread ay a time may take the lead
			pthread_mutex_lock(&mutex);
			
			//now passing the whole arguement struct 
			//and the index on which a request was found
			cache_insert(arguement, i);
			
			i++;
				

			//unlocking the mutex  
			pthread_mutex_unlock(&mutex);		
		}
	}
}

//~~~~~~~~~~~~~~~~~~~~~~~~~~~|M  A  I  N|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//
int main()
{
	//Opening the requests file to read the total requests
	int count, i ;
	FILE * probe;
	probe = fopen("requests.txt", "r");
	if( probe < 0 )
	{
		perror("[ERROR] TROUBLE OPENING FILE");
		return -1;
	}
	
	
	//Calling a custom function 
	//to get the sum of requests 
	//as well as the the requests as they are
	//and store them in a custom struct array
	count = get_Requests(probe);
	rewind(probe);
	t_Requests requests[count];
	memcpy( requests, convert_Req(probe, count), sizeof(requests) );
	for(i = 0; i < count; i++)
	{
		if( requests[i].id < 10 )
		{
			printf("\n[%d ] ", requests[i].id );
		}else
		{
			printf("\n[%d] ", requests[i].id );
		}
		
		printf(" |  [%d] ", requests[i].size );
	}
	
	
	//Declaring the variables to store the 
	//Documented Values such as 
	//[cache] . [ttl] . [threads]
	int cache1;
	int thread_Setting , ttl_Setting;
	ttl_Setting = get_Settings(&thread_Setting, &cache1);
	printf("\n[SETTING NUMBER OF TREADS TO] : %d", thread_Setting);
	printf("\n[SETTING CACHE CAPACITY TO] :%d ", cache1);
	printf("\n[SETTING TIME TO LIVE TO] :%d ", ttl_Setting);
	
			
	//Declaring vital parts
	//like the total threads
	//and two status variables
	//[check] . [status]
	pthread_t thread[thread_Setting + 1];	
	int check;
	void *  status;
	
	
	//In order to pass multiple arguements to [p_thread_create]
	//We use a struct [t_Arguement]
	t_Arguement arg;
	arg.request = requests;
	arg.cache = cache1;
	arg.sum = count;
	arg.index = 0;
	//Creating the proper amount of threads 
	//as the Document says making the programm completely modular
	for(i = 0; i < thread_Setting; i++)
	{		
			arg.id = i;
			pthread_create(&thread[i], NULL, handle_Request, &arg);	
	}
	
	//executing the p_thread_join in order to wait 
	//for them to finshi 
	for(i = 0; i < thread_Setting; i++)
	{
		
		pthread_join(thread[i], &status);
	}
	
	
				
	return 0;

}

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|F   I  L  E|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//
//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|F U N  C U T I O N S|~~~~~~~~~~~~~~~~~~~~~~~~~~//
//This function is used to count the number 
//of requests on the txt file
//a very useful function which heps to dynamically 
//allocate memory for the stuct array as seen 
int get_Requests( FILE * probe )
{
	
	
	char buffer[MAX_LINE];
	const char spc[2] = " ";
	int id, size, count, i;
	char* token; 
	
	fgets(buffer, MAX_LINE, probe);
	count = 0;
	while( fgets(buffer, MAX_LINE, probe) )
	{
		count++;			
	}

	return count;
}

//This function seperates the requests found on the 
//text file based on the observation that they are 
//differentiated by a blank space ' '
//so the [strtok()] function is applied
t_Requests * convert_Req( FILE * probe, int count)
{
	t_Requests * requests = malloc( sizeof(t_Requests) * count );
	const char * spc = " ";
	char * token;
	char buffer[MAX_LINE];
	int i = 0;
	
	fgets(buffer, MAX_LINE, probe);
	printf("\n[ID]  |  [SIZE]");
	printf("\n------|--------");
	
	
	
	while( fgets(buffer, MAX_LINE, probe) )
	{
		int cast;
		//thus the id stored in [requets.id]
		token = strtok(buffer, spc); 
		requests[i].id = atoi(token);
		
		//and the size in [requests.size]
		token = strtok(NULL, spc);
		requests[i].size = atoi(token);
		
		
		i++;
	}
	return requests;
}


//This function uses the the previous logic but to isolate
//the integer part of the txt file 
//and asigning it to the proper variable
//to get the system specifications
int get_Settings(int * thread_Setting, int * cache_Setting)
{
	FILE * probe;
	char buffer[MAX_LINE];
	int flag = 0;
	probe = fopen("configuration.txt", "r");
	if( probe < 0 )
	{
		perror("\n[ERROR] TROUBLE OPENING FILE ");
		return -1;
	}
	printf("\n");
	while(  fgets(buffer, MAX_LINE, probe) )
	{
		char * parse = buffer;
		char  setting[MAX_LINE];
		printf("[READING] %s", buffer);
		strcpy(setting, "");
		while( *parse  )
		{
			if( isdigit(*parse) )
			{
				strncat(setting, parse, 1);
				parse ++;
			}
			else
			{
				parse++;
			}	
		}
		if(flag == 0)
		{
			*thread_Setting = atoi(setting);
		}	
		else if(flag == 1)
		{
			*cache_Setting = atoi(setting);
		}			
		else 
		{
			return atoi(setting); 
		}	
		flag++;
	}
}
