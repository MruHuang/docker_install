
[ HackMd ](https://hackmd.io/@pONMMCWcRMSASTBgLGLkyg/mru_docker_install).

docker指令紀錄與說明
==========================

Created by mru huang, last modified on Dec 01, 2020

部屬
==

1.  docker stack rm mmrm
    
    1.  把mmrm開頭的docker _Container_  全部先關掉
        
2.  docker network ls
    
    1.  經聽docker 網路的情況，可以知道服務是不是已經全部都關閉了
        
3.  export $(cat mmrm\_env)
    
    1.  把設定檔引用進linux裡面準備引用
        
4.  docker stack deploy -c mmrm-docker-compose.yml mmrm
    
    1.  把compose跑起來，建置新的docker環境，並以mmrm作為命名開頭
        
5.  docker ps
    
    1.  觀察_Container_  是不是都已經建立起來
        
6.  docker service ls
    
    1.  觀察服務是不是都正常
        
7.  可以砍tasks.db(如果無法啟動docker)
    
    1.  /var/lib/docker/swarm/worker
        
8.  `docker swarm join-token worker`
    
    1.  取黑箱node派發給其他機器的token
        
    2.  進入白箱設定
        
        1.  docker swarm join \\ --token <token> \\ <myvm ip>:<port>
            
        2.  docker swarm join --token SWMTKN-1-35g90mpj6tmnvd5wj2kyiipu6p9taq55r7ar9ebhemdsse3zf4-15ys9dpt7z5084vzpfsf18eb5 172.105.209.27:2377
            
    3.  取名稱
        
        1.  docker node update --label-add name=mmrm\_core CORE\_NODE\_ID(node的ID，\*的或Leader)
            
        2.  docker node update --label-add name=mmrm\_extension EXTENSION\_NODE\_ID(node的ID)
            

進入docker容器
==========

1.  docker exec -it 容器id或名稱 bash
    
    1.  進入指定的docker _Container_  內部並使用bash的操作介面
        
2.  exit
    
    1.  離開docker _Container_ 
        

啟用work
======

1.  docker exec -it work的id supervisorctl
    
    1.  進入work介面
        
2.  reread
    
    1.  重新讀取work的相關設定資料
        
3.  update
    
    1.  重啟work
        

mmrm-docker-compose.yml講解
=========================

### version

要使用dovker的版本

`version: '3.7'`

### network

1.  建立全部服務會使用到的網路溝通名稱
    
2.  該網路經由指定後可以讓docker _Container_  互相溝通
    

networks:
  core\_frontend:
  core\_backend:
  core\_rabbitmq:
  core\_mysql\_replica:
  core\_api:
  extension\_frontend:
  extension\_backend:
  extension\_api:
  app\_api\_backend:
  app\_api\_redis:
  pos\_api\_backend:
  files\_api:
  push\_backend:
  push\_api:
  pos\_api:
  extension\_mysql:
  member\_mysql:
  wallet\_backend:

### volumes

建立資料夾，該資料夾在各專案間可以共享，可以指定從哪掛載

volumes:
  # redis:
    # driver: local
  phpmyadmin:
    driver: local
  # elasticsearch:
    # driver: local

### service

要啟動的服務，相關服務的設定，都寫在這，以下為各個設定的解說

1.  `core_workspace`
    
    1.  服務名稱
        
2.  `image`
    
    1.  該服務的映像黨位子
        
    2.  可以在dockerhub上分享
        
    3.  [https://hub.docker.com/](https://hub.docker.com/r/wishmobile/mmrm/tags)
        
    4.  `wishmobile/mmrm:core_workspace`
        
        1.  這個就是在dockerhub上的`wishmobile/mmrm`的專案裡面的`core_workspace`的設定黨
            
3.  `deploy`
    
    1.  部屬節點設定
        
    2.  在分散部屬於不同機器上，可以利用這個設定做分群部屬
        
4.  `volumes`
    
    1.  使用的資料夾建立
        
        1.  來源 : 存放位子 : 啟用額外服務
            
    2.  `- ${CORE_SOURCE_PATH}:/var/www:cached`
        
        1.  讀env黨的位子 : 放進/var/www : 啟用cached機制(增加讀檔速度與程式上的cached無關)
            
5.  `environment`
    
    1.  變更環境變數
        

`core_workspace`:服務名稱

services:
## MMRM\_Core Workspace Utilities ##################################
    core\_workspace:
      image: wishmobile/mmrm:core\_workspace
      deploy:
        mode: replicated
        replicas: 1
        placement:
          constraints:
            - node.labels.name == mmrm\_core
        restart\_policy:
          condition: on-failure
      networks:
        - core\_frontend
        - core\_backend
        - core\_rabbitmq
        - core\_api
        - app\_api\_redis
        - files\_api
        - push\_api
        - extension\_mysql
        - member\_mysql
      volumes:
        - ${CORE\_SOURCE\_PATH}:/var/www:cached

docker服務本身狀態檢查
==============

dockerd
