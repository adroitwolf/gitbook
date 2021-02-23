---
title: qt TCP UDP-多线程笔记
date: 2019-05-08 08:40:06
categories:
- c++
tags:
- qt
- TCP UDP
---

# qt TCP UDP 多线程解决并发思路



## TCP解决思路

-----

目的：每一个客户端连接都需要QTCPSocket开辟一条新的线程

解决方法：

1. 分别继承QTCPServer和QTCPSocket来分别实现Server和Socket（我这里是mTCPServer继承QTCPServer,mTCPSoket继承QTCPSokcet）

2. mTCPServer重写incomingConnection来实现socket的自动连接，其实就是不需要connect等待连接，直接进入函数
``` c++
protected:
    void incomingConnection(qintptr handle);
```
![](https://s2.ax1x.com/2019/05/09/Ecyj3Q.png)
3. 重写incomingConnection()函数

```c++
void MyTcpServer::incomingConnection( qint32 socketDescriptor)
{
    qDebug()<<tr("有新的连接 :-socketDescriptor-")<<socketDescriptor;
	
    emit displayAccount(true); //这里是我给主界面发信号,有用户连接
    MyTcpSocket *tcptemp = new MyTcpSocket(socketDescriptor);

	//初始化线程
    QThread *thread = new QThread(tcptemp);
    
    //收到客户端发送的信息
    connect(tcptemp,&MyTcpSocket::receiveData,this,&MyTcpServer::receiveDataSlot);
    //客户端断开链接
    connect(tcptemp,&MyTcpSocket::socketDisconnect,this,&MyTcpServer::disconnectSlot);
    //客户端断开链接 关闭线程
    connect(tcptemp,&MyTcpSocket::disconnected,thread,&QThread::quit);
    //向socket发送信息
    
    // 发送注册信息
    connect(this,&MyTcpServer::sendRegisterData,tcptemp,&MyTcpSocket::sendRegisterData);

    
    //将socket移动到子线程运行
    tcptemp->moveToThread(thread);
    thread->start();
    
    //将Qthread放到Map控制
    clients->insert(socketDescriptor,tcptemp);

    qDebug()<<tr("目前客户端数量：")<<clients->size();
}
```

4. mTCPSokcet解决相应的信号即可


## 多线程UDP解决思路

----

UDP的话就比较简单了,最核心的代码是这个

```c++

/**
 * function:监听端口
 * @brief MyUDPServer::startService
 */
void MyUDPServer::startService()
{
    //这里监听端口
    this->mUdpSocket = new QUdpSocket(this);
    int error =this->mUdpSocket->bind(QHostAddress::Any,9999);
    qDebug()<<error;
    QObject::connect(mUdpSocket,SIGNAL(readyRead()),this,SLOT(readData()));
}

MyUDPServer::MyUDPServer(QObject *parent):QObject(parent)
{

}
/**
 * function: 多线程读取数据
 * @brief MyUDPServer::readData
 */
void MyUDPServer::readData()
{
    //获取到传来的数据
    while(this->mUdpSocket->hasPendingDatagrams()){
        //获取数据
        QByteArray array;
        QHostAddress address;
        quint16 port;
   		//根据可读数据来设置空间大小
        array.resize(this->mUdpSocket->pendingDatagramSize());
        this->mUdpSocket->readDatagram(array.data(),array.size(),&address,&port); //读取数据
      
        //创建线程
        MsgAction *msgAction = new MsgAction(array,address,port);
        QThread *thread = new QThread(msgAction);
        //需要封装

        qDebug()<<QThread::currentThread();
        //移动到线程中
        msgAction->moveToThread(thread);

        thread->start();
        msgAction->run();
       //发送反馈数据
    }

}
```

