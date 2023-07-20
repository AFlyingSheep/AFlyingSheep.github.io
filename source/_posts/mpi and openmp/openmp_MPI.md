---
title: HPC基础：MPI+OPENMP
date: 2022-08-05 20:38:06
tags:
- [HPC]
categories: 
- [HPC]
- [CS]
mathjax: true
description: 高性能计算的编程基础，基于C++语言，入门HPC的第一站。
top_img: /image/covers/picture/picture1.jpg
cover: /image/covers/hpc.png
---

# MPI

## 第一个并行程序

```c++
// 1. 第一个MPI程序
int hello_world(int argc, char* argv[]) {
    char greeting[MAX_STRING];
    int comm_sz; // 进程核数
    int my_rank; // 进程等级

    // 进行所有必要的初始化设置，调用该函数前不得调用其他MPI函数
    // 同样存在MPI_Finalize()与其对应，在他们中间可以使用MPI函数
    MPI_Init(NULL, NULL);

    // Init的一个目的为用户启动程序时定义由用户启动所有进程所组成的通信子，成为MPI_COMM_WORLD
    // 注：下面两个函数第一个参数为通信子，第二个为输出

    // 返回通信子进程数
    MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);
    // 返回正在调用进程在通信子中的进程号
    MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);

    // 消息发送与接收
    // 消息发送
    /*
    int MPI_Send(
        void* msg_buf_p      , //指向包含信息内容的指针
        int   msg_size       ,
        MPI_Datatype msg_type, //数据量
        int   dest           , //目的
        int   tag            , //标签
        MPI_Comm communicator);//通信子
    */
    // 消息接收
    /*
    int MPI_Recv(
        void* msg_buf_p      ,
        int   buf_size       ,
        MPI_Datatype         ,
        int   source         ,
        int   tag            ,
        MPI_Comm communicator,
        MPI_Status* status_p //是一个输出！
        );
    )
    */
    // 当recv_type = send_type && recv_buf_sz >= send_buf_sz，则可以被接受
    // 接收的SOURCE核TAG有通配符，但通信子没有。

    // 0号进程用于接收信息并打印
    if (my_rank == 0) {
        printf("I'm main process, greetings from process %d of %d!\n", my_rank, comm_sz);
        for (int i = 1; i < comm_sz; i++) {
            // 使用MPI_ANY_SOURCE保证按完成工作顺序打印,MPI_ANY_TAG同理
            //MPI_Recv(greeting, MAX_STRING, MPI_CHAR, i, 0, MPI_COMM_WORLD, MPI_STATUSES_IGNORE);
            MPI_Recv(greeting, MAX_STRING, MPI_CHAR, MPI_ANY_SOURCE, 0, MPI_COMM_WORLD, MPI_STATUSES_IGNORE);
            printf("%s\n", greeting);
        }
    }
    // 其他进程用于发送信息
    else {
        // 输出信息到字符串greeting中
        sprintf_s(greeting, "Greetings from process %d of %d!", my_rank, comm_sz);
        // 将消息发送给0号进程
        MPI_Send(greeting, strlen(greeting) + 1, MPI_CHAR, 0, 0, MPI_COMM_WORLD);
    }

    // MPI_Status 结构体，对其成员访问分别获取source和tag信息：
    // status.MPI_SOURCE
    // status.MPI_TAG
    // 通过MPI_Get_count(&status, recv_type, &count)获取到接受元素的数量（返回到count中）


    MPI_Finalize();
    return 0;
}
```

## 利用MPI实现梯形积分（点对点通信）

```c++
// 2. 利用MPI实现梯形积分（点对点通信）
/*
    设计一个并行程序：
    1. 将问题划分为多个任务
    2. 在任务间识别出需要的通信信道
    3. 将任务聚合为复合任务
    4. 在核上分配任务
*/
double this_function(double x) {
    return x;
}
/*
    输入：只允许0号进程进行输入
    输出：可能乱序，需要自己进行编写
*/

double Trap(double left, double right, double n, double h) {
    double res = this_function(left) / 2 + this_function(right) / 2;
    for (int i = 1; i <= n - 1; i++) {
        res += this_function(left + h * i);
    }
    res = res * h;
    return res;
}

int Trapezoids (int argc, char* argv[]) {
    int my_rank, comm_sz, n;
    MPI_Init(NULL, NULL);
    int left, right;
    
    MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);
    MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);

    // 0号进程进行输入
    if (my_rank == 0) {
        cout << "Input a, b and n\n";
        cin >> left >> right >> n;
        for (int dest = 1; dest < comm_sz; dest++) {
            MPI_Send(&left, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
            MPI_Send(&right, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
            MPI_Send(&n, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
        }
    }
    // 其他进程读取输入
    else {
        MPI_Recv(&left, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        MPI_Recv(&right, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        MPI_Recv(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    }

    // 计算负责区域的左右区间、分段数量
    int h = (right - left) / n;
    int local_n = n / comm_sz;

    int local_left = left + my_rank * h * local_n;
    int local_right = right + my_rank * h * local_n;

    // 在负责区域进行串行计算
    double local_res = Trap(local_left, local_right, local_n, h);
    double total_res = local_res;

    // 返回结果
    if (my_rank != 0) {
        MPI_Send(&local_res, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD);
        cout << "I'm process " << my_rank << ", I send the result";
    }
    else {
        for (int i = 1; i < comm_sz; i++) {
            MPI_Recv(&local_res, 1, MPI_DOUBLE, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            total_res += local_res;
        }
    }
    // 0号进程输出结果
    if (my_rank == 0) {
        printf("With n = %d trapezoids, our estimate\n", n);
        printf("of the integral from %f to %f = %.15e\n", left, right, total_res);
    }
    MPI_Finalize();
    return 0;
}
```

## 集合通信

```c++
/*
    集合通信：设计通信子中所有进程的通信函数
    点对点通信：MPI_Send类似

    一些集合通信的注意事项：
    1. 通信子内所有进程必须调用相同的集合通信函数
    2. 参数必须相容，如dest_process都应该相同
    3. output_data_p只作用于dest_process
    4. 集合通信函数只通过通信子和调用顺序进行匹配！！！调用顺序很重要！P69表3-3

*/

// 集合通信解决梯形积分
/*
int MPI_Reduce(
    void*           input_data_p,
    void*           output_data_p,
    int             count,
    MPI_Datatype    datatype,
    MPI_Op          operator,
    int             dest_process,
    MPI_Comm        comm
)
    MPI_Allreduce 将计算的结果返回给所有进程，参数表同Reduce，但是没有dest
*/

/*
    广播函数：MPI_Bcast(data_p(in/out), count, data_type, source_proc, comm)
*/
// 使用广播函数获取输入并分发
void Get_input(int my_rank, int comm_sz, double* a_p, double* b_p, int* n_p) {
    if (my_rank == 0) {
        printf("Input a, b and n.\n");
        scanf_s("%lf%lf%d", a_p, b_p, n_p);
    }
    MPI_Bcast(a_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
    MPI_Bcast(b_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
    MPI_Bcast(n_p, 1, MPI_INT, 0, MPI_COMM_WORLD);
}

/*
    散射：MPI_Scatter()
    限制：块划分法+块大小相同
*/
// 读取向量并划分分发
// 以下仅考虑可以整除的情况，习题3.13将讨论不可整除
void Read_vector(double local_a[], int local_n, int n, 
    char vector_name[], int my_rank, MPI_Comm comm) {
    double* a = NULL;
    if (my_rank == 0) {
        a = (double*) malloc(n * sizeof(double));
        for (int i = 0; i < n; i++) {
            scanf_s("%lf", &a[i]);
        }
        MPI_Scatter(a, local_n, MPI_DOUBLE, local_a, local_n, MPI_DOUBLE, 0, comm);
        free(a);
    }
    else {
        MPI_Scatter(a, local_n, MPI_DOUBLE, local_a, local_n, MPI_DOUBLE, 0, comm);
    }
}

/*
    聚集：MPI_Gather()
    注：recv_count 指的是每个进程接收到的数据量
    限制：块划分法+块大小相同
*/
void Print_vector(double local_b[], int local_n, int n, char title[], int my_rank, MPI_Comm comm) {

    double* a = (double*)malloc(n * sizeof(double));

    if (my_rank == 0) {
        MPI_Gather(local_b, local_n, MPI_DOUBLE, a, local_n, MPI_DOUBLE, 0, MPI_COMM_WORLD);
        printf("%s\n", title);
        for (int i = 0; i < n; i++) printf("%lf ", a[i]);
        printf("\n");
        free(a);
    }
    else {
        MPI_Gather(local_b, local_n, MPI_DOUBLE, a, local_n, MPI_DOUBLE, 0, MPI_COMM_WORLD);
    }
}

/*
    全局聚集：MPI_Allgather()：相当于MPI_Gather + MPI_Bcast
    将每个进程Send_p的内容串联起来放到Recv_p中
*/
// 矩阵与向量相乘
void Matirx_Mul_Vector(double local_A[], double local_x[], double local_y[], 
    int local_m, int n,int local_n, int my_rank, MPI_Comm comm) {
    
    double* x;

    x = (double*)malloc(n * sizeof(double));

    // 因为此处，函数得到的是划分后的x，但是运算需要整个x向量，故需要Allgather进行向量补全
    MPI_Allgather(local_x, local_n, MPI_DOUBLE, x, local_n, MPI_DOUBLE, comm);

    for (int local_i = 0; local_i < local_m; local_i++) {
        local_y[local_i] = 0.0;
        for (int j = 0; j < n; j++)
            local_y[local_i] += local_A[local_i * n + j] * x[j];
    }
    free(x);
}
```

## 信息整合

```c++
/*
    MPI提供的三个整合多条消息数据的手段：
    1. count
    2. 派生数据类型
    3. pack/unpack
*/

/* 派生数据类型：如果发送数据的函数知道数据项的类型以及在内存中数据项集合的相对位置，
   就可以在数据项被发送出去之前在内存中将数据项聚集起来。
   组成：数据类型 + 偏移

   派生数据类型更像是将每一个指定位移的数据的偏移量记录下来，每次通信的时候传输这些数据
*/
// 创建派生数据类型
/*
int MPI_Type_create_struct(
    int count,                          // 数据类型中的元素个数，下面各个参数数组都有count个元素
    int array_of_blocklengths[],        // 允许单独数据项为数组或子数组，如第一个元素是一个含5个元素的数组，则aob[0] = 5
    MPI_Aint array_of_displacements[],  // 距离消息起始位置的偏移量,单位为字节
    MPI_Datatype array_of_types[],      // 数据类型
    MPI_Datatype* new_type_p    (out)
    )
*/
// 使用前需要指定
/*
int MPI_Type_commit(MPI_Datatype* new_type_p);
*/

// 使用派生数据类型的Get_input
/*
原：
void Get_input(int my_rank, int comm_sz, double* a_p, double* b_p, int* n_p) {
    if (my_rank == 0) {
        printf("Input a, b and n.\n");
        scanf_s("%lf%lf%d", a_p, b_p, n_p);
    }
    MPI_Bcast(a_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
    MPI_Bcast(b_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
    MPI_Bcast(n_p, 1, MPI_INT, 0, MPI_COMM_WORLD);
}
*/
/*
    修改后
*/
void Build_mpi_type(double* a_p, double* b_p, int* n_p, MPI_Datatype* data_type) {
    int array_of_blocklengths[3] = { 1, 1, 1 };
    MPI_Datatype array_of_types[3] = { MPI_DOUBLE, MPI_DOUBLE, MPI_INT };
    MPI_Aint a_addr, b_addr, n_addr;
    MPI_Aint array_of_addr[3] = { 0 };

    MPI_Get_address(a_p, &a_addr);
    MPI_Get_address(b_p, &b_addr);
    MPI_Get_address(n_p, &n_addr);

    array_of_addr[1] = b_addr - a_addr;
    array_of_addr[2] = n_addr - b_addr;

    MPI_Type_create_struct(3, array_of_blocklengths, array_of_addr, array_of_types, data_type);
    MPI_Type_commit(data_type);
}

void Get_input_new(int my_rank, int comm_sz, double* a_p, double* b_p, int* n_p) {
    MPI_Datatype input_mpi_t;

    Build_mpi_type(a_p, b_p, n_p, &input_mpi_t);

    if (my_rank == 0) {
        printf("Input a, b and n.\n");
        scanf_s("%lf%lf%d", a_p, b_p, n_p);
    }
    
    MPI_Bcast(a_p, 1, input_mpi_t, 0, MPI_COMM_WORLD);

    MPI_Type_free(&input_mpi_t);
}
```

## 并行排序算法——分布式算法实现

```c++
// 冒泡排序变种——奇偶交换排序
// 找寻交换进程号
void ComputePartner(int my_rank, int comm_sz, int phase, int& partner) {
    if (phase % 2 == 0) {
        if (my_rank % 2 == 0)
            partner = my_rank + 1;
        else partner = my_rank - 1;
    }
    else {
        if (my_rank % 2 == 0)
            partner = my_rank - 1;
        else partner = my_rank + 1;
    }
    if (partner == -1 || partner == comm_sz)
        // 注：MPI_PROC_NULL作为源进程或目标进程进程号时，调用通信函数直接返回，不产生通信
        partner = MPI_PROC_NULL;

}

// 注：MPI_Send有两种发送方式，如果数据量较大可能会造成阻塞发送，可能导致死锁。
// 问题：程序安全如何保证？使用MPI_Ssend（表示同步，发送如果没被接受便阻塞）
// 问题：怎么修改奇偶排序使其安全？重构通信。
// MPI_Sendrecv() 阻塞式发送接收，可以保证安全
// MPI_Sendrecv_replace() 发送和接受使用的是同一个缓冲区

// 奇偶冒泡
void Merge_low(int my_keys[], int recv_keys[], int temp_keys[], int local_n, bool flag) {

    int m_i = 0, r_i = 0, t_i = 0;
    if (!flag) {
        m_i = local_n - 1;
        r_i = local_n - 1;
    }

    // flag为true时，交换代表前一个线程（取小），反之取大。
    // 每次前一线程取小，后一线程取大
    while (t_i < local_n) {
        if (flag) {
            if (my_keys[m_i] < recv_keys[r_i]) {
                temp_keys[t_i] = my_keys[m_i];
                t_i++; m_i++;
            }
            else {
                temp_keys[t_i] = recv_keys[r_i];
                t_i++; r_i++;
            }
        }
        else {
            if (my_keys[m_i] > recv_keys[r_i]) {
                temp_keys[t_i] = my_keys[m_i];
                t_i++; m_i--;
            }
            else {
                temp_keys[t_i] = recv_keys[r_i];
                t_i++; r_i--;
            }
        }
    }
    if (flag) for (m_i = 0; m_i < local_n; m_i++) my_keys[m_i] = temp_keys[m_i];
    else for (m_i = local_n - 1; m_i >= 0; m_i--) my_keys[local_n - m_i - 1] = temp_keys[m_i];
}


int compare(const void* e1, const void* e2)
{
    //将void*类型的指针e1和e2强制类型转换成int*型
    return  *((int*)e1) - *((int*)e2);
    //一定要强制类型转换，因为e1和e2都是void*指针，没有类型的指针
    //如果不想要升序排列，想要按降序排列，就可以return  *((int*)e2) - *((int *)e1);
}

void My_Sort(int argc, char* argv[]) {
    int my_rank, comm_sz, n;
    int* array = NULL;
    MPI_Init(NULL, NULL);

    MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);
    MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);

    // 进程0读入
    if (my_rank == 0) {
        cout << "Input size of array:";
        cin >> n;

        array = (int*)malloc(sizeof(int) * n);
        

        for (int i = 0; i < n; i++) {
            cin >> array[i];
        }

    }
    // 广播总个数
    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    // local_n代表每个进程分配的元素个数
    int local_n = n / comm_sz;
    // cout << "local_n" << local_n << endl;
    int* local_array = (int*)malloc(sizeof(int) * local_n);
    // if (my_rank == 0) for (int i = 0; i < local_n; i++) local_array[i] = array[i];

    // 分配元素
    MPI_Scatter(array, local_n, MPI_INT, local_array, local_n, MPI_INT, 0, MPI_COMM_WORLD);
    //cout << "Process" << my_rank << " 's array is " << local_array[0] << endl;

    for (int i = 0; i < 101; i++) {

        int* local_receive = (int*)malloc(sizeof(int) * local_n);
        int* local_temp = (int*)malloc(sizeof(int) * local_n);
        
        int partner;

        // 计算本回合交换目标
        ComputePartner(my_rank, comm_sz, i, partner);

        // cout << "epoch" << i << " " << "Process" << my_rank << " 's partner is " << partner << endl;

        // 本进程元素排序
        qsort(local_array, local_n, sizeof(local_array[0]), compare);

        // 互发元素元素
        MPI_Sendrecv(local_array, local_n, MPI_INT, partner, 0, 
            local_receive, local_n, MPI_INT, partner, 0, MPI_COMM_WORLD, MPI_STATUSES_IGNORE);

        // cout << "epoch" << i << " " << "Process" << my_rank << " 's recv is " << local_receive[0] << "and" << local_receive[1] << endl;

        bool flag = (i % 2 == 0) && (my_rank % 2 == 0) || (i % 2 == 1) && (my_rank % 2 == 1);

        if (partner != MPI_PROC_NULL)
            Merge_low(local_array, local_receive, local_temp, local_n, flag);

        // cout << "epoch" << i << " " << "Process" << my_rank << " 's sorted array is " << local_array[0] << "and" << local_array[1] << endl;

        free(local_receive);
        free(local_temp);
    }

    // 收集最后结果
    MPI_Gather(local_array, local_n, MPI_INT, array, local_n, MPI_INT, 0, MPI_COMM_WORLD);

    if (my_rank == 0)
        for (int i = 0; i < n; i++) cout << array[i] << " ";
    MPI_Finalize();
}

```

# OpenMP

## 第一个OpenMP程序

```c++
#include<stdio.h>
#include<stdlib.h>
#ifdef _OPENMP
#   include<omp.h>
#endif

void Hello_world(void);

int main(int argc, char* argv[]) {
    /* 得到线程数 */
    int thread_count = strtol(argv[1], NULL, 10);

    // #pragma 开头，代表预处理器指令，如果不支持pragma的编译器会忽略该指令
    // parallel代表结构化代码块，是一条C语句或一个入口和一个出口的复合C语句
    // parallel指令添加num_threads子句(子句是修改指令的文本)
#   pragma omp parallel num_threads(thread_count)
    Hello_world();
    /*
        执行parallel指令后，thread_count - 1个线程被启动，
        原始线程成为主线程master,额外线程称为从线程slave
        master + slave称为线程组，线程组每个线程都执行parallel后的代码块
        该处存在一个隐式路障，当所有线程执行完代码块，从线程终止，主线程才继续执行
    */
    return 0;
}

void Hello_world() {
    // 得到自己的线程编号
    int my_rank = omp_get_thread_num();
    // 得到线程组中的线程数
    int thread_count = omp_get_num_threads();
    printf("Hello world from thread %d of %d\n", my_rank, thread_count);
    return;
}
```

## OpenMP实现梯形积分法

```c++
#include<stdio.h>
#include<stdlib.h>
#ifdef _OPENMP
#   include<omp.h>
#endif

void Trap(double a, double b, int n, 
    double* global_result_p);
double f(double x) {
    double y = x;
    return y;
}

int main(int argc, char* argv[]) {
    double global_result = 0.0;
    double a, b;
    int n;
    int thread_count = strtol(argv[1], NULL, 10);

    printf("Input a, b and n\n");
    scanf("%lf %lf %d", &a, &b, &n);

#   pragma omp parallel num_threads(thread_count)
    Trap(a, b, n, &global_result);

    printf("Estimate of the integral = %.14e\n", global_result);
    return 0;
}

void Trap(double a, double b, int n, 
    double* global_result_p) {
    // 注意，计算并没有检查n是否能被thread_num整除，如不能需要另加修改
    double h, x, my_result;
    double local_a, local_b;
    int i, local_n;

    int my_rank = omp_get_thread_num();
    int thread_count = omp_get_num_threads();

    h = (b - a) / n;
    local_n = n / thread_count;
    local_a = a + my_rank * local_n * h;
    local_b = local_a + h * local_n;

    my_result = (f(local_a) + f(local_b)) / 2.0;
    for (int i = 1; i <= local_n - 1; i++) {
        x = local_a + i * h;
        my_result += f(x);
    }
    my_result *= h;

#   pragma omp critical
    *global_result_p += my_result;
}

// 注意：在parallel指令前已经被声明的变量拥有在线程组中共享作用域，
//      而在块中声明的变量（如函数中的局部变量）有私有作用域

// 加入规约子句的Trap版本

// 我们更加习惯于不使用指针来传递临界变量：
double Local_trap(double a, double b, int n);
// 而在调用的时候，会改变parallel块如下：
/*
double global_result = 0.0;
# pragma omp parallel num_threads(thread.count)
{
    double my_result = 0.0;
    my_result += Local_trap(double a, double b, int n);
#   pragma omp critical
    global_result += my_result;
}
*/

// OpenMP提供归约(将相同的归约操作符重复应用到操作数序列)操作符，
// 所有操作的中间结果储存在归约变量中
// 于是我们修改parallel块如下：
/*
double global_result = 0.0;
# pragma omp parallel num_threads(thread.count) \
    reduction(+: global_result)
{
    global_result += Local_trap(double a, double b, int n);
}
*/
// reduction子句的语法：reduction(<operator>: <variable list>)
// operator可以为：+ * - & | ^ && ||
// 注意：
// 1. - 不满足交换律和结合律，OpenMP不能保证正确运行
// 2. 归约变量如果为double or float，浮点数运算不满足结合律，可能会有不同
// 3. reduction中包含的变量是共享的，但是每个线程都会创建自己的私有变量（初始化为0或1（*）等情况）
//    当parallel块结束时会将私有变量的值整合到共享变量中

// parallel for指令
// OpenMP提供parallel for 指令，能够并行化串行积分
// 串行积分改进：
void Serial_Trap(double h, double a, double b, double approx, int n) {
    h = (b - a) / n;
    approx = (f(a) + f(b)) / 2.0;
#   pragma omp parallel for num_threads(thread_count) \
        reduction(+: approx)
    for (int i = 1; i <= n - 1; i++)
        approx += f(a + i * h);
    approx = h * approx;
}
// parallel for与parallel指令非常不同，
// 在parallel指令之前的块，其工作必须由线程本身在线程之间划分
// parallel for指令缺省划分方式由系统决定
// approx必须作为归约变量，否则approx += 将变成无保护临界区
// 注意：parallel for中循环变量缺省作用域是私有的

// 对于parallel for的警告
/*
    1. 只会并行化for循环，不会并行化while, do-while
    2. OpenMP只能并行化确定迭代次数的for循环
        - 由for语句本身确定
        - 在循环执行前确定
    3. OpenMP只能并行化典型结构for循环
        - index必须是整形或指针类型
        - start, end, incr必须有一个兼容类型
        - start, end, incr在循环执行期间不改变
        - index只能由for语句中增量表达式修改
    4. 唯一例外：循环体中可以有一个exit调用
*/

// 数据依赖性
/*
    OpenMP编译器不检查被parallel for指令并行化的循环所包含的迭代间依赖关系
    当使用parallel for指令时要寻找循环依赖
*/
```

## parallel for的数据、循环依赖

以估算$\pi$为例：

$\pi=4[1-\dfrac{1}{3}+\dfrac{1}{5}-\dfrac{1}{7}+\dots]=4\sum^{\infty}_{k=0}\dfrac{(-1)^k}{2k+1}$

```c++
// 数据依赖性
/*
    OpenMP编译器不检查被parallel for指令并行化的循环所包含的迭代间依赖关系
    当使用parallel for指令时要寻找循环依赖
*/
#include<stdio.h>
#include<stdlib.h>
#ifdef _OPENMP
#   include<omp.h>
#endif

void serial_pai(int thread_count, int n);
void parallel_pai_1_0(int thread_count, int n);
void parallel_pai_1_1(int thread_count, int n);
void parallel_pai(int thread_count, int n);

int main(int argc, char* argv[]) {
    int thread_count = strtol(argv[1], NULL, 10);
    // serial_pai(thread_count, 1000);
    // parallel_pai_1_0(thread_count, 1000);
    // parallel_pai_1_1(thread_count, 1000);
    parallel_pai(thread_count, 1000);
    return 0;
}
// 串行代码
void serial_pai(int thread_count, int n) {
    double factor = 1.0;
    double sum = 0.0;
    for (int k = 0; k < n; k++) {
        sum += factor / (2 * k + 1);
        factor = -factor;
    }
    double pi_approx = 4.0 * sum;
    printf("pai: %lf\n", pi_approx);
}
// 并行1.0
void parallel_pai_1_0(int thread_count,  int n) {
    double factor = 1.0;
    double sum = 0.0;
#   pragma omp parallel for num_threads(thread_count)\
        reduction(+: sum)
    for (int k = 0; k < n; k++) {
        sum += factor / (2 * k + 1);
        // !!!该处存在数据依赖性，若k次迭代在一个线程，k+1次在另一个线程，
        // 会导致factor值错误
        factor = -factor;
    }
    double pi_approx = 4.0 * sum;
    printf("pai: %lf\n", pi_approx);
}

// 并行1.1
void parallel_pai_1_1(int thread_count, int n) {
    double factor = 1.0;
    double sum = 0.0;
#   pragma omp parallel for num_threads(thread_count)\
        reduction(+: sum)
    for (int k = 0; k < n; k++) {
        factor = (k % 2 == 0) ? 1.0 : -1.0;
        sum += factor / (2 * k + 1);
        // factor = -factor;
    }
    double pi_approx = 4.0 * sum;
    printf("pai: %lf\n", pi_approx);
}
// 问题：
// 注意，缺省情况下任何在循环前声明的变量在线程间都是共享的
// 因此factor是共享的，可能存在线程间修改导致错误，因此要保证factor私有作用域
// 最终版本
void parallel_pai(int thread_count, int n) {
    double factor = 1.0;
    double sum = 0.0;
#   pragma omp parallel for num_threads(thread_count)\
        reduction(+: sum) private(factor)
    for (int k = 0; k < n; k++) {
        factor = (k % 2 == 0) ? 1.0 : -1.0;
        sum += factor / (2 * k + 1);
    }
    double pi_approx = 4.0 * sum;
    printf("pai: %lf\n", pi_approx);
}

// Openmp提供一个子句default(none),要求用户明确每个变量的作用域
// 如对pai的计算可以修改为：
/*
#   pragma omp parallel for num_threads(thread_count)\
        default(none) reduction(+: sum) private(k, factor)\
        shared(n)
*/
```

## OpenMP实现奇偶排序

```c++
#include<stdio.h>
#include<stdlib.h>
// #ifdef _OPENMP
#   include<omp.h>
// #endif

void serial_odd_even_order(int thread_count, int a[], int n);
void parallel_odd_even_order(int thread_count, int a[], int n);
void parallel_odd_even_order_2(int thread_count, int a[], int n);
void swap(int &a, int &b) {
    int t = a;
    a = b;
    b = t;
}

int main(int argc, char* argv[]) {
    int a[10] = {2, 1, 16, 8, 6, 0, 7, 11, 12, 15};
    int thread_num = strtol(argv[1], NULL, 10);
    parallel_odd_even_order_2(thread_num, a, 10);
    for (int i = 0; i < 10; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void serial_odd_even_order(int thread_count, int a[], int n) {
    int phase;
    for (phase = 0; phase < n; phase++) {
        if (phase % 2 == 0) {
            for (int i = 1; i < n; i += 2) {
                if (a[i - 1] > a[i]) 
                    swap(a[i - 1], a[i]);
            }
        }
        else {
            for (int i = 1; i < n - 1; i++) {
                if (a[i] > a[i + 1]) {
                    swap(a[i], a[i + 1]);
                }
            }
        }
    }
}

// parallel 1.0
// 我们发现，最外层的for循环具有循环依赖，并不适合并行化
void parallel_odd_even_order(int thread_count, int a[], int n) {
    int phase;
    for (phase = 0; phase < n; phase++) {
        if (phase % 2 == 0) {
#           pragma omp parallel for num_threads(thread_count) \
                default(none) shared(a, n)
            for (int i = 1; i < n; i += 2) {
                if (a[i - 1] > a[i]) 
                    swap(a[i - 1], a[i]);
            }
        }
        else {
#           pragma omp parallel for num_threads(thread_count) \
                default(none) shared(a, n)
            for (int i = 1; i < n - 1; i++) {
                if (a[i] > a[i + 1]) {
                    swap(a[i], a[i + 1]);
                }
            }
        }
    }
}

// parallel 2.0
// parallel每进行一次外部循环都会创建和合并线程，产生开销
// 每次执行内部循环都会使用相同数量的线程，因此我们希望只创建一次线程，并在每次内部循环执行中重用
void parallel_odd_even_order_2(int thread_count, int a[], int n) {
    int phase;
#   pragma omp parallel num_threads(thread_count) \
        default(none) shared(a, n) private(phase)
    for (phase = 0; phase < n; phase++) {
        if (phase % 2 == 0) {
#           pragma omp for
            for (int i = 1; i < n; i += 2) {
                if (a[i - 1] > a[i]) 
                    swap(a[i - 1], a[i]);
            }
        }
        else {
#           pragma omp for
            for (int i = 1; i < n - 1; i++) {
                if (a[i] > a[i + 1]) {
                    swap(a[i], a[i + 1]);
                }
            }
        }
    }
}
```

## 线程调度子句schedule

```c++
#include<stdio.h>
#include<stdlib.h>
#ifdef _OPENMP
#   include<omp.h>
#endif
#include<time.h>
#include<math.h>

double function(int i) {
    int j, start = i * (i + 1) / 2, finish = start + i;
    double return_val = 0.0;

    for (j = start; j <= finish; j++) {
        return_val += sin(j);
    }
    return return_val;
}

int main(int argc, char *argv[]) {
    // parallel for只是粗略的使用块分割，如果调用函数与需要时间成正比(function())
    // 这样的分配方式显然不佳。于是我们使用schedule子句实现好的迭代分配

    // 下面是对于几种schedule的尝试

    // 无调度
    int thread_count = strtol(argv[1], NULL, 10);
    double result_sum = 0.0;
    int n = 100;
    double start_time = (double) clock();
#   pragma opt parallel for num_threads(thread_count) \
        reduction(+: result_sum) shared(n)
    for (int i = 0; i <= n; i++) {
        result_sum += function(i);
    }
    double end_time = (double) clock();
    printf("No schedule, time is %.4lf, result is %.4lf\n", end_time - start_time, result_sum);
    
    // dynamic调度
    result_sum = 0.0;
    start_time = (double) clock();
    #   pragma opt parallel for num_threads(thread_count) \
        reduction(+: result_sum) shared(n) schedule(dynamic)
    for (int i = 0; i <= n; i++) {
        result_sum += function(i);
    }
     end_time = (double) clock();
    printf("Dynamic schedule, time is %.4lf, result is %.4lf\n", end_time - start_time, result_sum);
    
    // guided调度
    result_sum = 0.0;
    start_time = (double) clock();
    #   pragma opt parallel for num_threads(thread_count) \
        reduction(+: result_sum) shared(n) schedule(guided)
    for (int i = 0; i <= n; i++) {
        result_sum += function(i);
    }
     end_time = (double) clock();
    printf("Guided schedule, time is %.4lf, result is %.4lf\n", end_time - start_time, result_sum);

    // runtime调度
    result_sum = 0.0;
    start_time = (double) clock();
    #   pragma opt parallel for num_threads(thread_count) \
        reduction(+: result_sum) shared(n) schedule(runtime)
    for (int i = 0; i <= n; i++) {
        result_sum += function(i);
    }
     end_time = (double) clock();
    printf("Runtime schedule, time is %.4lf, result is %.4lf\n", end_time - start_time, result_sum);
    
    // 自动调度
    result_sum = 0.0;
    start_time = (double) clock();
    #   pragma opt parallel for num_threads(thread_count) \
        reduction(+: result_sum) shared(n) schedule(auto)
    for (int i = 0; i <= n; i++) {
        result_sum += function(i);
    }
     end_time = (double) clock();
    printf("Auto schedule, time is %.4lf, result is %.4lf\n", end_time - start_time, result_sum);
    

    return 0;
}

// eg. 三个线程，12个任务
// 调度方式：
// 1. static: 以轮转方式分配chunksize个线程给每个线程（chunksize 默认近似为total_iterations / thread_count)
// chunksize = 2
// Thread0: 0 1 6 7
// Thread1: 2 3 8 9
// Thread2: 4 5 10 11

// 2. dynamic: 迭代被分为chunksize个连续块，每个线程执行一块，执行完再向系统申请(chunksize默认为1)
// 3. guided: 类似于dynamic, 每个线程执行完一块后，向系统请求另一块，但是新块会变小
// guided如下图
// 4. runtime: 需要自己配置环境变量指定

// 调度选择
/*
    系统开销: guided > dynamic > static
    1. 如果每次迭代计算量几乎相同，默认调度方式性能最好
    2. 若计算量线性递增或递减，采用比较小的chunksize的static比较好
    3. 如果每次迭代事先不确定，需要进行调参，可以通过配置环境变量来比较不同调度策略下性能
*/
```

![1](/image/openmp/1.png)

## 小结

![1](/image/openmp/2.png)

![1](/image/openmp/3.png)

![1](/image/openmp/4.png)

![1](/image/openmp/5.png)

![1](/image/openmp/6.png)

