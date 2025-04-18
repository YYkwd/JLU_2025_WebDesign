<template>
    <div 
      ref="scrollContainer" 
      style="height: 80vh; overflow-y: auto; display: flex; flex-wrap: wrap; gap: 20px; margin-top: 20px;"
    >
      <el-card 
        v-for="(item, index) in orders" 
        :key="item.id" 
        shadow="hover"
        style="width: 300px;"
      >
      <img :src="item.photo" alt="商品图片" style="width: 100%; height: 200px; object-fit: cover;" />
        <h3 style="margin: 10px 0;">订单编号: {{ item.id }}</h3>
        <p>商品ID: {{ item.goodsId }}</p>
        <p>商品名称: {{ item.goodsName }}</p>
        <p>购买数量: {{ item.count }} 件</p>
        <p>总价格: {{ item.totalPrice }} 元</p>
        <p>到达时间: {{ item.arriveTime }}</p>
        <p>创建时间: {{ item.createTime }}</p>
        <p  style="color: blue; font-weight: bold; font-size: 16px; letter-spacing: 1px;">
          配送中 | 骑手: {{ item.deliverName }}
        </p>
      </el-card>
      
      <el-loading v-if="loadingMore" text="加载中..." />
    </div>
  </template>
  
  <script lang="ts" setup>
  import { ref, onMounted } from 'vue';
  import { ElMessage } from 'element-plus';
  import api from '@/api/request'; // 导入封装好的axios实例
  import { useUserStore } from '@/store/user'; // 调用用户的store获取authorization
  
  const UserStore = useUserStore(); // 获取用户的authorization
  
  // 定义订单接口
  interface OrderItem {
    id: number;            // 订单编号
    goodsId: number;       //商品编号
    count: number;         //待送商品数目
    totalPrice: number;    //总价格
    arriveTime: string;    //订单送达时间
    createTime: string;    //订单创建时间
    purchaseId: number;    //购买者ID
    sellId: number;        //卖家ID
    status: number;        //订单状态
    deliverName: string;    //配送骑手昵称
    photo: string ;    //订单商品照片
    goodsName: string;    //商品名称
  }
  
  // 定义 orders 为 OrderItem 数组类型
  const orders = ref<OrderItem[]>([]);
  const loadingMore = ref(false); // 是否正在加载更多
  
  // 定义 scrollContainer 用于绑定滚动容器
  const scrollContainer = ref<HTMLElement | null>(null);
  
  // 获取订单数据
  const fetchOrders = async () => {
    try {
      loadingMore.value = true;
      // 获取所有订单数据的API请求
      const response = await api.get('/user/orders', {
        headers: {
          authorization: UserStore.authorization // 获取用户的token
        }
      });
  
      const data = response.data.data;
      
      // 过滤 status 为 1 的订单 // 获取正在配送的订单
      const filteredOrders = data.filter((order: OrderItem) => order.status === 1);
      console.log("配送中的订单" ,filteredOrders );
      if (filteredOrders.length > 0) {
        orders.value = filteredOrders; // 将过滤后的订单赋值给 orders
      } else {
        ElMessage.info('没有待配送的订单');
      }
  
      loadingMore.value = false;
    } catch (error) {
      console.error('获取订单信息失败：', error);
      ElMessage.error('获取订单信息失败');
      loadingMore.value = false;
    }
  };
  
  
  // 初始化加载订单
  onMounted(() => {
    fetchOrders(); // 第一次加载订单
  });
  </script>
  
  <style scoped>
  .el-card {
    margin-bottom: 20px;
  }
  
  .el-button {
    margin-top: 10px;
  }
  
  .el-loading {
    text-align: center;
    width: 100%;
    margin-top: 20px;
  }
  </style>
  