---
title: "Linked List"
categories: [Storage, Data Structure]
tags: [Storage, DS]
use_math: true
---

  > 연결리스트

#### 연결리스트란?
  - 서로 연결되어 있지 않은 값들을 **임의로 연결**하여 배열과 같은 역할을 수행하는 자료구조
  - 구성요소
    - 데이터를 담는 변수
    - 특정 노드를 연결하는 포인터 변수

#### 종류
##### 단일 연결리스트
  - head부터 시작하여 tail을 향해서만 탐색 가능한 연결리스트

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #define ERR -99999
    
    // making stack using linked list
    
    typedef struct Node {
    	int key;
    	struct Node *next;
    } ND;
    
    // function that creates new node
    // returns pointer of new node
    ND *createNewNode(int value) {
    	ND *temp = (ND *)malloc(sizeof(ND));
    	if (!temp) { // defensive coding
    		return NULL;
    	}
    	temp->key = value;
    	return temp; // node return
    }
    
    // insert new node to linked list
    // return 0 if successfully inserted, else return 1
    int insert(ND **head, int value) {
    	ND *temp = createNewNode(value);
    	if (!temp) // can't create new node
    		return 1;
    	ND *cur = *head,
    		*pre = NULL;
    	if (!*head) { // no node in linked list
    		*head = temp;
    		return 0;
    	}
    	while (cur) // find tail of linked list
    		pre = cur, cur = cur->next;
    	pre->next = temp; // link new node to the tail of linked list
    	return 0;
    }
    
    int delete (ND **head) {
    	if (!(*head)) { // no node in linked list
    		printf("No node!");
    		return ERR;
    	}
    	else if (!(*head)->next) { // only 1 node in linked list
    		ND *temp = *head;
    		*head = NULL; // no node in linked list
    		int retval = temp->key;
    		free(temp);	   // delete node
    		return retval; // return node's key value
    	}
    	ND *pre = NULL,
    		*cur = *head;
    	while (cur->next) // find tail of linked list
    		pre = cur, cur = cur->next;
    	ND *temp = cur;	  // tail of linked list
    	pre->next = NULL; // cut link
    	int retval = temp->key;
    	free(temp); // delete node
    	return retval;
    }
    
    int main() {
    	ND *head = NULL; // head of linked list
    	/*
    	using some code
    	*/
    	return 0;
    }
    ```

##### 원형 연결리스트
  - head에서 시작하여 단방향으로 다시 돌아올 수 있는 연결리스트
##### 이중 연결리스트
  - head에서 시작하여, 앞뒤로 탐색 가능한 연결리스트



#### 소스코드

여러 가지로 응용 가능

```c
/* myLinkedList.c */
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
	int value;
	struct Node *next;
} ND;
typedef ND *NDPtr;

NDPtr createNewNode(int value);				  // create a Node
void insertToHead(NDPtr *head, NDPtr node);	  // insert node to First
void insertToMiddle(NDPtr *head, NDPtr node); // insert node to Middle
void insertToTail(NDPtr *head, NDPtr node);	  // insert node to Tail
void deleteFromHead(NDPtr *head);			  // delete node from head
void deleteFromMiddle(NDPtr *head);			  // delete node from middle
void deleteFromTail(NDPtr *head);			  // delete node from tail
NDPtr findTail(NDPtr head);					  // find Tail of List
NDPtr search(NDPtr head, int value);		  // search the value
void changeValue(NDPtr head);				  // change node's value to input
int isEmpty(NDPtr head);					  // 1 if Empty, 0 if not Empty
void printList(NDPtr head);					  // print All List
void instructions();						  // how to use this system
int getInput();								  // get input type int

int main() {
	NDPtr Head = NULL;
	int choose, end = 1, temp;
	while (end) {
		instructions();
		scanf("%d", &choose);
		switch (choose) {
			case 1:
				insertToHead(&Head, createNewNode(getInput()));
				break;
			case 2:
				insertToMiddle(&Head, createNewNode(getInput()));
				break;
			case 3:
				insertToTail(&Head, createNewNode(getInput()));
				break;
			case 4:
				deleteFromHead(&Head);
				break;
			case 5:
				deleteFromMiddle(&Head);
				break;
			case 6:
				deleteFromTail(&Head);
				break;
			case 7:
				temp = getInput();
				if (search(Head, temp))
					printf("%d found!\n", temp);
				else
					printf("%d not found!\n", temp);
				break;
			case 8:
				printList(Head);
				break;
			case 9:
				changeValue(Head);
				break;
			case 0:
				end = 0;
				break;
		}
	}
	printf("End of System");
	char str;
	scanf("\n%c", &str);
	return 0;
}

NDPtr createNewNode(int value) { // create a Node
	NDPtr temp = (NDPtr)malloc(sizeof(ND));
	if (temp) {
		temp->value = value;
		temp->next = NULL;
	}
	return temp;
}

void insertToHead(NDPtr *head, NDPtr node) { // change fisrt node to Node
	node->next = *head;
	*head = node;
	return;
}

void insertToMiddle(NDPtr *head, NDPtr node) {
	if (isEmpty(*head)) {
		printf("List is Empty!\n");
		free(node);
		return;
	}
	NDPtr temp = search(*head, getInput());
	if (!temp) {
		printf("Target Not Found!\n");
		free(node);
	}
	else {
		node->next = temp->next;
		temp->next = node;
	}
	return;
}

void insertToTail(NDPtr *head, NDPtr node) { // 맨 처음에 삽입이 안되는 문제가 있음
	if (isEmpty(*head))
		*head = node;
	else
		findTail(*head)->next = node;
	return;
}

void deleteFromHead(NDPtr *head) {
	if (isEmpty(*head)) {
		printf("List is Empty!\n");
		return;
	}
	NDPtr temp = *head;
	*head = temp->next;
	free(temp);
	return;
}

void deleteFromMiddle(NDPtr *head) {
	if (isEmpty(*head)) {
		printf("List is Empty!\n");
		return;
	}
	int a = getInput();
	if (search(*head, a)) {
		NDPtr temp = *head;
		if (temp->value == a) {
			deleteFromHead(head);
			return;
		}
		while (temp->next->value != a)
			temp = temp->next;
		NDPtr temp2 = temp->next;
		temp->next = temp2->next;
		free(temp2);
	}
	else
		printf("Target not Founded!\n");
	return;
}

void deleteFromTail(NDPtr *head) {
	if (isEmpty(*head)) {
		printf("List is Empty!\n");
		return;
	}
	NDPtr temp = *head;
	if (!temp->next) {
		free(temp);
		*head = NULL;
		return;
	}
	while (temp->next->next)
		temp = temp->next;
	NDPtr temp2 = temp->next;
	temp->next = NULL;
	free(temp2);
	return;
}

NDPtr findTail(NDPtr head) { // find Tail of list
	NDPtr temp = head;
	while (temp->next)
		temp = temp->next;
	return temp;
}

NDPtr search(NDPtr head, int value) { // find node with value in list
	NDPtr temp = head;
	while (temp && temp->value != value)
		temp = temp->next;
	return temp;
}

void changeValue(NDPtr head) {
	NDPtr temp = search(head, getInput());
	if (temp)
		temp->value = getInput();
	else
		printf("Can't find that value!\n");
	return;
}

int isEmpty(NDPtr head) { // 1 if Empty, 0 if not Empty
	if (head)
		return 0;
	else
		return 1;
}

void printList(NDPtr head) {
	NDPtr temp = head;
	if (isEmpty(head)) {
		printf("List is Empty!\n");
		return;
	}
	while (temp) {
		printf("%d -> ", temp->value);
		temp = temp->next;
	}
	printf("NULL\n");
	return;
}

void instructions() {
	printf("1 : 리스트 맨 앞에 삽입\n"
		"2 : 리스트 중간에 삽입\n"
		"3 : 리스트 맨 뒤에 삽입\n"
		"4 : 리스트 맨 앞 노드 삭제\n"
		"5 : 리스트 중간 노드 삭제\n"
		"6 : 리스트 맨 뒤 노드 삭제\n"
		"7 : 리스트 값 검색\n"
		"8 : 리스트 전체출력\n"
		"9 : 리스트 특정 값 수정\n"
		"0 : 종료\n"
		"Choose? ");
	return;
}

int getInput() {
	int temp;
	printf("Input a Data :: ");
	scanf("%d", &temp);
	return temp;
}
```



#### 기본 구현 방법
  1. 시작점 포인터를 생성 ( head )
  2. 노드를 동적할당으로 만들어 목적에 맞게 연결시킴 ( 앞 / 중간 / 뒤 )
  3. 노드를 삭제할 때는 연결을 재설정 한 후 **free** 해주어야 함 ( **메모리 누수** 우려 )

#### 참고
  - 동적할당