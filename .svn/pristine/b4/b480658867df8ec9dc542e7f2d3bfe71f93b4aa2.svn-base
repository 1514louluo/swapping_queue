/*
*	A simple swapping queue implementation. <ring double linklist implement the queue, just one producer and one consumer>
*	License: MIT
* 	Copyright (C) 2014 louluo <18915413902@163.com> if there is some bug, please let me know!
*/
#ifndef _SWAP_QUEUE_H
#define _SWAP_QUEUE_H

#include <unistd.h>
#include <pthread.h>
#include <semaphore.h>
#include <errno.h>
#include <time.h>

#include <stdlib.h>
//#include "jemalloc"
#define P_MALLOC malloc      /* here can use jemalloc, improve performance 1/3 */
#define P_FREE   free        /* here can use jemalloc, improve performance 1/3 */

#define COND_TIMEOUT 3       /* semp try wait cond timeout, here is magic number, you can resize it yourself */

/* encapsulate for one function only one return */
#define	return_val_if(condition, return_value) 							\
do{											 	\
  if((condition))										\
    {												\
      return (return_value);									\
    }												\
}while(0)

#define	return_val_if_fail(condition, return_value) 						\
do{											 	\
  if(!(condition))										\
    {												\
      return (return_value);									\
    }												\
}while(0)

#define	return_if(condition) 									\
do{											 	\
  if((condition))										\
    {												\
      return;											\
    }												\
}while(0)


#define	return_if_fail(condition) 								\
do{											 	\
  if(!(condition))										\
    {												\
      return;											\
    }												\
}while(0)

#ifdef __cplusplus
extern "C"
{
#endif

typedef void (*pfree_handler)(void *);

struct swap_queue_node
{
	struct  swap_queue_node             *next;        /* swap linklist node */
	struct  swap_queue_node             *prev;        /* swap linklist node */
	void                                *data;        /* data, elem is a pointer */
};

struct swap_queue_head
{
	struct swap_queue_node              *in;          /* producer */
	struct swap_queue_node              *out;         /* consumer */
	pthread_mutex_t                     *lock;        /* swap in and out lock */
	sem_t                               *no_empty;    /* producer is empty? */
};

extern struct swap_queue_head *swap_queue_init(pthread_mutex_t *lock, sem_t *cond);
extern void swap_queue_free(struct swap_queue_head *head, pfree_handler handler);
extern int swap_queue_put(struct swap_queue_head *head, void *data);
extern void *swap_queue_get(struct swap_queue_head *head);

#ifdef __cplusplus
}
#endif //__cplusplus
#endif /*_SWAP_QUEUE_H*/
