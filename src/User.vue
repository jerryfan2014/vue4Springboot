<script setup>
    import {ref} from "vue"
    import axios from "axios"

    //定义同步变量 
    const userData = ref([])
    // 新增用户的临时数据
    const newUser = ref({id:null,  name: null, age: null, email: null, avatar:null });
    let action="add"
    // 图片上传
    const fileInput = ref(null);
    const selectedFile = ref(null);
    const previewUrl = ref(null);
    const uploadStatus = ref('');
    
    // 获取用户列表
    function listUser(){
        // axios.get("http://localhost:80/user/list")
        // 使用请求代理路径， 解决CORS跨域问题  
        axios.get("/api/user/list")
        .then(result => {
            // 注意：Axios 的响应对象 (result) 本身也有一层默认的 data 属性，
            userData.value = result.data.data
        })
        .catch(err => {
            console.log(err)
        })
    }

    //更新用户
    const updateUser = (userId) => {
        action = "update"
        // 获取用户信息       
        axios.get("/api/user/get/" + userId)
        .then(response => {            
            // newUser.value = response.data.data;
            // alert(JSON.stringify(response.data))
            newUser.value = response.data.data;            
            // alert("newUser.value:" + JSON.stringify(newUser.value))
            showAddUserDiv.value = true;
        })
        .catch(err => {
            console.log(err)
            alert(response.data.msg)
        })
    };

    // 保存（新增或更新用户）
    const saveUser = () => {
        // ...展开运算符，将数组展开为参数列表
        // const id = userData.value.length ? Math.max(...userData.value.map(user => user.id)) + 1 : 1;        
        //id非自增, 也不要赋值，由后端生成后传入前端
        // userData.value.push({ id, ...newUser.value });   
        axios.post("/api/user/save", newUser.value, {
                headers: {
                    "Content-Type": "application/json", 
                }, 
            })
        .then(response => {
            // 恢复初始值
            newUser.value = {id:null,  name: null, age: null, email: null, avatar:null };
            showAddUserDiv.value = false;

            // 这里有 2 种方式处理某条数据被更新后的 列表更新
            //刷新用户列表
            listUser();
        })
        .catch(err => {
            console.log(err)
        })        
    };

    // 控制新增/更新用户信息的div的显示或隐藏
    const showAddUserDiv = ref(false);    
    // 显示新增/更新div
    const addUser = () => {
        action = "add"
        showAddUserDiv.value = true;    
    };

    // 隐藏新增/更新div， 同时清空所有文本框
    const cancelAddUser = () => {
        newUser.value = {id:null,  name: null, age: null, email: null, avatar:null };
        showAddUserDiv.value = false;        
    };

    // 删除用户
    const deleteUser = (id) => {
        if (!confirm('你确定要删除数据吗？')) {
            return;
        }

        // axios.post("/api/user/delete/${id}", newUser.value)
        // 也可以使用反引号嵌入变量'/api/user/del/${id}', 请求协议与后端保持一致（后端是DeleteMapping)
        axios.delete('/api/user/del/' + id)
        .then(response => {
            console.log(response.data)
        })
        .catch(err => {
            console.log(err)
        })  

        userData.value = userData.value.filter((user) => user.id != id);
    };

    const searchUser = ref({ name: '', age: ''});
    // 条件查询
    const doSearch = (userName, userAge) => {
        // 这里可以根据 userName 和 userAge 做过滤或者发请求查询
        //console.log('查询用户:', userName.value, userAge.value);
        axios.get("/api/user/search", {params: {name: userName, age: userAge}})
        .then(result => {
            userData.value = result.data.data
        })
        .catch(err => {
            console.log(err)
        })
    };
    

//--------------------------------- 上传图片--------------------------
// 响应式数据


// 处理文件选择
const handleFileChange = (e) => {
  const file = e.target.files[0];
  if (!file) return;

  selectedFile.value = file;
  
  // 生成预览图（可选）
  previewUrl.value = URL.createObjectURL(file);
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    uploadStatus.value = '请先选择图片！';
    // alert("请选择图片")
    return;
  }

  const formData = new FormData();
  formData.append('file', selectedFile.value);   // 文件字段
  formData.append('userId', newUser.value.id);
  
  try {
    uploadStatus.value = '上传中...';
    // const response = await axios.post('/api/user/uploadUserPhoto', formData, {
    //   headers: {
    //     'Content-Type': 'multipart/form-data'    // 
    //   }
    // });
    console.log(JSON.stringify(formData))
    const response = await axios.post('/api/user/uploadUserPhoto', formData);
    uploadStatus.value = '上传成功！';
    console.log('服务器响应:', response.data);
    
    // 清空选择和预览（可选）
    fileInput.value.value = '';
    selectedFile.value = null;
  } catch (error) {
    uploadStatus.value = '上传失败: ' + error.message;
    // alert('上传失败: ' + error.message)
  }
};
//--------------------------------- 上传图片--------------------------

    //触发获取用户列表
    listUser();
</script>

<template>
    <div>        
        <label>用户名: </label>包含
        <el-input v-model="searchUser.email" style="width: 240px" placeholder="请输入姓名" />&nbsp;&nbsp;
        <label>年龄: </label>大于
        <el-input v-model="searchUser.age" style="width: 240px" placeholder="请输入年龄" />
        &nbsp;&nbsp;&nbsp;&nbsp;
        <el-button type="primary" @click="doSearch(searchUser.name, searchUser.age)">查询</el-button>&nbsp;&nbsp;
        <el-button type="primary" @click="addUser()">新增</el-button> 
    </div> 
    <br>
    <div id="tableDiv">
    <!-- 传统 html table
        <table border="1" cellpadding="10">
            <tr>
                <td >userId</td>
                <td >userName</td>
                <td >userAge</td>
                <td >email</td>
                <td >操作</td>
            </tr>
            <tr v-for = "user in userData">
                <td>{{user.id}}</td>
                <td>{{user.name}}</td>
                <td>{{user.age}}</td>
                <td>{{user.email}}</td>
                <td><button @click="addUser()" >新增</button> &nbsp;&nbsp; <button @click="deleteUser(user.id)" >&nbsp;删除&nbsp;</button></td>
            </tr>
        </table>            
        -->
        <!-- 使用element-plus -->
        <el-table :data="userData" stripe border>
            <el-table-column prop="id" label="ID" width="120"/>
            <el-table-column prop="name" label="姓名" width="120"/>
            <el-table-column prop="age" label="年龄" width="120"/>
            <el-table-column prop="email" label="邮箱" width="220"/>
            <el-table-column width="180" label="操作">                
                <template #default="{row}">
                    <el-button type="primary" @click="updateUser(row.id)">修改</el-button>&nbsp;&nbsp; 
                    <el-button type="primary" @click="deleteUser(row.id)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>

<div v-if="showAddUserDiv" class="modal" style="height: 700px">
  <div class="modal-content" style="display: flex; width: 800px;">
    <!-- 左边表单区域 -->
    <div style="flex: 1; padding: 20px;">
      <span class="close" @click="cancelAddUser">&times;</span>
      <h3>
        <p v-if="action == 'add'">新增用户</p>
        <p v-else>更新用户</p>
      </h3>
      <div style="margin-bottom: 10px;">        
        <label>姓名:</label> 
        <el-input v-model="newUser.name" style="width: 240px" placeholder="请输入姓名" />
      </div>
      <div style="margin-bottom: 10px;">
        <label>年龄:</label> 
        <el-input v-model="newUser.age" style="width: 240px" placeholder="请输入年龄" />
      </div>
      <div style="margin-bottom: 10px;">
        <label>邮箱:</label> 
        <el-input v-model="newUser.email" style="width: 240px" placeholder="请输入邮箱" />
      </div>
      <br>
      <el-button type="primary" @click="saveUser">保存</el-button>&nbsp;&nbsp;
      <el-button type="primary" @click="cancelAddUser">取消</el-button>
    </div>
    
    <!-- 右边图片区域 -->
    <div style="flex: 1; padding: 20px; display: flex; flex-direction: column; align-items: center; justify-content: center;">
      <h3>用户头像</h3>
        <!--
      <div style="width: 200px; height: 200px; border: 1px dashed #ccc; display: flex; align-items: center; justify-content: center; margin-bottom: 20px;">
        <img :src="newUser.avatar || '默认图片URL'" style="max-width: 100%; max-height: 100%;" />
      </div>
      <el-input       type="file"  @change="handleFileChange"  accept="image/*"   ref="fileInput"/>
      <el-button type="primary" @click="uploadImage">上传图片</el-button>      
      -->
      <div>
        <!-- 图片预览 -->
        <div style="width: 200px; height: 200px; border: 1px dashed #ccc; 
            display: flex; align-items: center; justify-content: center; 
            margin-bottom: 20px; overflow: hidden;">
            <img 
                v-if="previewUrl || newUser.avatar"
                :src="previewUrl || newUser.avatar" 
                alt="预览" 
                style="max-width: 100%; max-height: 100%; object-fit: contain;"
            />
        </div>
        <!-- 文件选择输入框 -->
        <input 
        type="file" 
        @change="handleFileChange" 
        accept="image/*"
        ref="fileInput"
        />
        <button @click="uploadImage">上传图片</button>
        <!-- 显示上传状态 -->
        <p v-if="uploadStatus">{{ uploadStatus }}</p>
    </div>

    </div>
  </div>
</div>
</template>

<style scoped>
    /* 模态对话框的背景样式 */
    .modal {
        display: block;
        position: fixed;
        z-index: 1;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        overflow: auto;
        background-color: rgb(0, 0, 0);
        background-color: rgba(0, 0, 0, 0.4);
    }

    /* 模态对话框内容样式 */
    .modal-content {
        background-color: #fefefe;
        margin: 15% auto;
        padding: 20px;
        border: 1px solid #888;
        width: 80%;
        max-width: 500px;
        border-radius: 10px;
        position: relative;
    }

   /* 关闭按钮样式 */
    .close {
        color: #aaa;
        position: absolute;  /* 改为绝对定位 */
        top: 10px;          /* 距离顶部10px */
        right: 20px;        /* 距离右侧20px */
        font-size: 28px;
        font-weight: bold;
        z-index: 1;         /* 确保按钮在最上层 */
        transition: color 0.2s; /* 添加颜色过渡动画 */
    }

    .close:hover,
    .close:focus {
        color: black;       /* 悬停时变黑色 */
        text-decoration: none;
        cursor: pointer;
}
</style>
