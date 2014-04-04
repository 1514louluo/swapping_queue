/*
*	A simple swapping queue implementation. <ring swap linklist implement the queue, just one producer and one consumer>
*	License: MIT
* 	Copyright (C) 2014 louluo <18915413902@163.com> if there is some bug, please let me know!
*/
#include "swap_queue.h"

/*
*	__new_swap_queue_node - alloc new swap queue node
*	@data: pointer to put into the swap queue
*		
*	This function alloc a new swap queue node, return node or NULL
*/
static inline struct swap_queue_node *__new_swap_queue_node(void *data)
{
	struct swap_queue_node *node;
	node = (struct swap_queue_node *)P_MALLOC(sizeof(struct swap_queue_node));
	return_val_if_fail(node, NULL);

	node->next = node->prev = NULL;
	node->data = data;
	return node;
}

/*
*	__free_swap_queue_node - free swap queue node
*	@node: swap_queue_node to free
*
*	This function free the swap_queue_node pointer
*/
static inline void __free_swap_queue_node(struct swap_queue_node *node)
{
	return_if_fail(node);
	P_FREE(node);
}


/*
*	__new_swap_queue_node_dummy - alloc new swap queue dummy, dummy is a sentinel
*
*	This function alloc a new swap queue dummy, return dummy or NULL
*/
static inline struct swap_queue_node *__new_swap_queue_node_dummy(void)        /* dummy is a sentinel */
{
	struct swap_queue_node *dummy;
	dummy = (struct swap_queue_node *)P_MALLOC(sizeof(struct swap_queue_node));
	return_val_if_fail(dummy, NULL);

	dummy->next = dummy->prev = dummy;
	dummy->data = NULL;
	return dummy;
}

/*
*	swap_queue_init - init swap queue head
*	@lock: swap in and out lock, here is mutex (also can use spinlock, if u wish)
*	@cond: this is a sem to show than is in empty
*
*	This function init swap queue head, return head or NULL
*/
struct swap_queue_head *swap_queue_init(pthread_mutex_t *lock, sem_t *cond)
{
	struct swap_queue_head *head;

	head = (struct swap_queue_head *)P_MALLOC(sizeof(struct swap_queue_head));
	return_val_if_fail(head, NULL);

	head->in = __new_swap_queue_node_dummy();
	if(NULL == head->in)
	{
		P_FREE(head);
		return NULL;
	}
	head->out = __new_swap_queue_node_dummy();
	if(NULL == head->out)
	{
		P_FREE(head);
		return NULL;
	}
	head->lock = lock;
	head->no_empty = cond;

	return head;	
}

/*
*	__free_swap_queue_dummy - swap queue dummy free
*	@dummy: the dummy to be free
*
*	This function free swap queue
*/
static inline void __free_swap_queue_dummy(struct swap_queue_node *dummy)
{
	P_FREE(dummy);
}

/*
*	swap_queue_free - swap queue free
*	@head: the head to be free
*	@handler: this function pointer show how to free the data pointer in node
*
*	This function free swap queue
*/
void swap_queue_free(struct swap_queue_head *head, pfree_handler handler)
{
	/* first free producer */	
	struct swap_queue_node *in = head->in->next;
	while(in != head->in)
	{
		struct swap_queue_node *tmp = in;
		in = in->next;
		handler(tmp->data);
		P_FREE(tmp);
	}
	__free_swap_queue_dummy(in);
	/* second free consumer */
	struct swap_queue_node *out = head->out->next;
	while(out != head->out)
	{
		struct swap_queue_node *tmp = out;
		out = out->next;
		handler(tmp->data);
		P_FREE(tmp);
	}
	__free_swap_queue_dummy(out);
	/* free head */
	P_FREE(head);
}

/*
*	__swap_queue_put - swap queue put
*	@head: which head u want to put
*	@node: which node u want put to the head
*
*	This function put the node to the head, return 0 is success, -1 is error
*/
static inline int __swap_queue_put(struct swap_queue_head *head, struct swap_queue_node *node)
{
	struct swap_queue_node *in_head = head->in;
	return_val_if_fail(in_head, -1);

	node->next = in_head->next;
	in_head->next->prev = node;
	in_head->next = node;
	node->prev = in_head;

	if(in_head == node->next)               /* only when in has one elem(except dummy), must notify no empty */
		sem_post(head->no_empty);
	return 0;
}

/*
*	swap_queue_put - swap queue put
*	@head: which head u want to put
*	@data: which data pointer u want put to the head
*
*	This function put the data to the head, return 0 is success, -1 is error
*/
int swap_queue_put(struct swap_queue_head *head, void *data)
{
	int ret;
	return_val_if(!head || !data, -1);

	struct swap_queue_node *node = __new_swap_queue_node(data);
	return_val_if_fail(node, -1);

	pthread_mutex_lock(head->lock);
	ret = __swap_queue_put(head, node);
	pthread_mutex_unlock(head->lock);
	return ret;
}

/*
*	__make_timeout - make sem timewait timeout
*	@tsp: 
*	@seconds: how many seconds u want to wait?
*
*	This fuction make sem timewait timeout
*/
static inline void __make_timeout(struct timespec *tsp, int seconds)
{
	clock_gettime(CLOCK_REALTIME, tsp);
	tsp->tv_sec += seconds;
}

/*
*	__swap_queue_get - get a swap queue node, sure this is fifo
*	@head: which head u want to get?
*
*	This function get a swap queue node from a certain head, return node or NULL
*/

static inline struct swap_queue_node *__swap_queue_get(struct swap_queue_head *head)
{
	struct swap_queue_node *out_head = head->out;
	return_val_if_fail(out_head, NULL);

	/* empty? swap in and out : do nothing */
	if(out_head == out_head->next) 	
	{
		struct timespec tsp;
		__make_timeout(&tsp, COND_TIMEOUT);
		int ret = sem_timedwait(head->no_empty, &tsp); 
		return_val_if(ret, NULL);    			/* sem_timewait return 0 is success */

		pthread_mutex_lock(head->lock);
		/* swap in and out */
		head->out = head->in;
		head->in = out_head;
		/* now in is empty */

		pthread_mutex_unlock(head->lock);
	}
	out_head = head->out;
	/* empty? NULL : last */
	struct swap_queue_node *last = out_head->prev;
	return_val_if(last == out_head, NULL);

	last->prev->next = last->next;
	last->next->prev = last->prev;

	return last;
}

/*
*	swap_queue_get - get a swap queue node data, sure this is fifo
*	@head: which head u want to get?
*
*	This function get a swap queue node data from a certain head, return node or NULL
*/
void *swap_queue_get(struct swap_queue_head *head)
{
	return_val_if_fail(head, NULL);
	struct swap_queue_node *node = __swap_queue_get(head);
	return_val_if_fail(node, NULL);

	void *data = node->data;
	__free_swap_queue_node(node);
	return data;
}


