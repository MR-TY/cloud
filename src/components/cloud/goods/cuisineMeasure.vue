<template>
  <div>
    <div style="margin-bottom:0px;display: flex;">
    	<div style="flex: 1;">
    		<Form>
	    		<Form-item class="row-1 order-type" label="计量名称">
	    			<Input v-model="ordermeasurekName" placeholder="请输入内容..." style="width: 240px;"></Input>
		      </Form-item>
	      </Form>
    	</div>
    	<div style="margin-right: 62%;">
    		<Button type="primary" style="width: 88px;" @click="modelDictionaryid">搜索</Button>
    	</div>
      <div style="margin-right:16px">
      	 <button class="add" @click="add">+新建</button>
      </div>
      <div>
       	<button class="delAll" @click="deletereasonDictionaryall">批量删除{{getalldeletereasonDictionary}}</button>
      </div>
    </div>
    <Table @on-selection-change="handdeldictionarySelection" ref="tableData" stripe :columns="dictionariesSet" :data="dictionariesData" no-data-text="没有相关数据"></Table>
    <div style="margin-top: 16PX;">
    	<button class="delAll">导出Excel</button>
    </div>
    <div style="overflow: hidden;margin-top: 16px;">
      <div style="float: right;">
        <Page :total="totalResult" :current="currentPage" @on-change="changePage"></Page>
      </div>
    </div>
    <!--新建-->
    <Modal
      class="modal-wrapper"
      v-model="addDictionary"
      title="添加菜品计量单位"
      :loading="loading"
      @on-ok="asyncDictionary" width="420">
      <h4 class="modal-title"><b style="color: #ff4949;margin-right: 5px;">*</b>单位名称</h4>
      <Input v-model="shiftMetering" placeholder="请输入..." style="margin-top: 10px;"></Input>
		 
    </Modal>
    <Modal
      class="modal-wrapper"
      v-model="editDictionary"
      title="编辑菜品计量单位"
      :loading="loading"
      @on-ok="asyncDictionary" width="420">
      <h4 class="modal-title"><b style="color: #ff4949;margin-right: 5px;">*</b>单位名称</h4>
      <Input v-model="shiftMetering" placeholder="请输入..." style="margin-top: 10px;"></Input>	  
    </Modal>
  </div>
</template>
<script type="text/ecmascript-6">
  import * as api from '../../../vuex/api';
   /**
 * 把 100 以内的正整数转为字符串，并保证2位，缺的前面加 '0'
 * @param s 值为 0 到 99
 * @returns '00' 到 '99'
 */
  const padSecond = s => ('0' + s).substr(String(s).length - 1);
  const PAGESIZE = 10;
  export default {
    data() {
      return {
      	orderType: [],
        modelDictionary: '',
        dictionariesData: [],
        addDictionary:false,
        editDictionary:false,
        selection:'',//多选
        totalResult: 0,
        currentPage: 1,
        meteringStatesid:'',//
        ids:[],
        modelDictionaryadd:'',//新建id
        shiftMetering:'',
        loading:false,
        editDictionaryid:'',///
        ordermeasurekName:'',//计量名称
        remarkState:false,
        remarkStatesid:'',
        dictionariesSet: [
          {
            type: 'selection',
            width: 60,
            align: 'center'
          },
          {
            title: '序号',
            type: 'index',
            width: 60,
            align: 'center'
          },
          {
            title: '菜品计量备注名称',
            key: 'value'
          },
          {
            title:'状态',
            key:'status',
            render: (h, params) => {
              return h('span', params.row.status === 0 ? '开启' : '禁用')
            }
          },
           {
            title:'创建时间',
            render: (h, p) => {
			       let createDate = new Date(p.row.createTime);
		         let dateStr= createDate.getFullYear() + '-'
			              + padSecond(createDate.getMonth() + 1)
			              + '-'
			              + padSecond(createDate.getDate())
			              + ' '
			              + padSecond(createDate.getHours())
			              + ':'
			              + padSecond(createDate.getMinutes());
			            return h('span', dateStr);
			    }
          },
          {
            title: '操作',
            key: 'action',
            width: 150,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  style: {
                    padding:'0',
                    marginRight: '32px',
                    width: '40px',
                    height: '24px',
                    borderRadius: '2px',
                    border: '1px solid #20a0ff',
                    lineHeight: '24px',
                    textAlign: 'center',
                    color: '#20a0ff',
                    fontSize: '13px',
                    background:'#fff',
                    verticalAlign:'top'
                  },
                  on: {
                    click: () => {
                      this.edit(params.index)
                    }
                  }
                }, '编辑'),
                h('Button', {
                  style: {
                    padding:'0',
                    width: '40px',
                    height: '24px',
                    borderRadius: '2px',
                    border: '1px solid #ff4949',
                    lineHeight: '24px',
                    textAlign: 'center',
                    color: '#ff4949',
                    fontSize: '13px',
                    background:'#fff'
                  },
                  on: {
                    click: () => {
                      this.deletereasonDictionary(params.index)
                    }
                  }
                }, '删除')
              ]);
            }
          },
          {
            title: '状态切换',
            key: 'switch',
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('i-switch', {
                  props: {
                    value: params.row.status === 0
                  },
                  on: {
                    'on-change': () => {
                      this.changeMetering(params.index);
                    }
                  }
                }),
              ])
            }
          }
        ]
      };
    },
    created(){
      this.getDictionary();
      //this.getclassification();
    },
    computed: {
      getalldeletereasonDictionary() {
        if (this.selection.length > 0) {
          return this.selection.length;
        }
        return 0
      }
    },
    methods: {
    	getDictionary(){
	       	 this.$http.post(api.DICTIONARYSETTINGS + '/getDictionary',
	          JSON.stringify(this.getParam('getSet')))
	          .then(res => {
	            //console.log('================================================',res);
	            let response = res.body;
	             if (response.code === 0) {
	              if (response.data.list) {
	                  this.dictionariesData = response.data.list;
	              }
	              this.totalResult = response.data.totalResult;
	            }
	        })
        .catch(err => {
            console.log(err);
         })
     },
     delsetDictionary(){
	       	 this.$http.post(api.DICTIONARYSETTINGS + '/deleteDictionary',
	          JSON.stringify(this.getParam('deleteDictionary')))
	          .then(res => {
	           //console.log(res);
	            let response = res.body;
	             if (response.code === 0) {
		              this.loading = false;
		              this.getDictionary();
		              this.$Message.success('删除成功');
		           } else {
		              this.loading = false;
		              this.$Message.error('删除失败');
               }
	        })
        .catch(err => {
            console.log(err);
         })
     },
      addDictionaryData() {                            //新建接口
          this.$http.post(api.DICTIONARYSETTINGS + '/addDictionary',
            JSON.stringify(this.getParam('addDictionary')))
            .then(res => {
              let response = res.body;
              if (response.code === 0) {
                this.loading = false;
                this.$Message.success('创建成功');
                this.addDictionary = false;
                this.getDictionary();
              } else {
                this.$Message.error(response.msg);
                this.loading = false;
              }
            })
            .catch(err => {
              console.log(err)
            })
      },
      editDictionaryData() {                                  //编辑接口
          this.$http.post(api.DICTIONARYSETTINGS + '/updateDictionary',
            JSON.stringify(this.getParam('editDictionary')))
            .then(res => {
              //console.log(res);
              let response = res.body;
              if (response.code === 0) {
                this.loading = false;
                this.$Message.success('修改成功');
                this.editDictionary = false;
                this.getDictionary();
              } else {
                this.$Message.error(response.msg);
                this.loading = false;
              }
            })
            .catch(err => {
              console.log(err)
        })
      },
     asyncDictionary() {         // 弹窗中的确定按钮
        if (this.type === 'addDictionarys') {
          this.addDictionaryData();
          
        } else {
          this.editDictionaryData();
        }
      },
     add(){                        // 添加班次操作弹窗
        this.type = 'addDictionarys';
        this.addDictionary = true;
        this.shiftMetering = '';
        //this.shiftlabel = '';
      },
      edit(index) {             // 编辑操作弹窗
        this.type = 'edit';
        this.editDictionary = true;
        this.meteringStatesid = this.dictionariesData[index].id;
        this.shiftMetering = this.dictionariesData[index].value;
        //console.log('=====================================',this.shiftcategory)
        //this.shiftlabel = this.dictionariesData[index].value;
      },
     modelDictionaryid(index){
     	this.getDictionary();
     },
    changePage(page){
        this.currentPage = page;
        this.getDictionary();
      },
    deletereasonDictionary(index){  //单个删除
      	this.$Modal.confirm({
          title: '删除',
          content: '此操作将删除该分类，是否继续？',
          loading: true,
          onOk: () => {
              this.ids = [];
              this.ids.push(this.dictionariesData[index].id);
              //console.log('============================',this.ids);
            setTimeout(() => {
              this.delsetDictionary(index);
              this.$Modal.remove();
            }, 1000);
          },
          onCancel: () => {
            this.loading = false;
            this.$Modal.remove();
          }
        });
      	
      },
      deletereasonDictionaryall() {     //批量删除
        this.$Modal.confirm({
          title: '删除',
          content: '此操作将批量删除菜品，是否继续？',
          loading: true,
          onOk: () => {
            setTimeout(() => {
              this.delsetDictionary();
              this.selection = []; //批量删除后，重置选中数量
              this.$Modal.remove();
            }, 1000);
          },
          onCancel: () => {
            this.loading = false;
            this.$Modal.remove();
          }
        });
      },
      handdeldictionarySelection(selection){
        this.selection = selection;
        console.log(this.selection);
        //this.delId = [];
        this.ids = [];
        for (let i = 0; i < this.selection.length; i++) {
           this.ids.push(this.dictionariesData[i].id);
        }
      }, 
      changeMetering(index){   //开关切换
      	if (this.dictionariesData[index].status === 0) {
          this.$Modal.confirm({
            title: '停用',
            content: '此操作将停用该备足名称，是否继续？',
            loading: true,
            onOk: () => {
              setTimeout(() => {
                this.dictionariesData.status = 1;
                this.meteringStatesid = this.dictionariesData[index].id;
                this.remarkState = false;
                this.editDictionaryData();
                this.$Modal.remove();
              }, 1000);
            },
            onCancel: () => {
            	this.getDictionary();
            }
          });
        } else {
          this.dictionariesData.status = 0;
          this.remarkStatesid = this.dictionariesData[index].id;
          this.remarkState = true;
          this.editDictionaryData();
        }
      }, 
     getParam(name) {
        if (name === 'getSet') {
          return {
            reqId: 0,
            channel: 0,
            os: "web",
            ver: "v2",
            appVer: "v2",
            model: "string",
            lng: "0",
            lat: "0",
            signType: "string",
            sign: "string",
            token: localStorage.getItem('token'),
            param: {
            	page: {
	                pageSize: PAGESIZE,
	                currentPage: this.currentPage
                },
	               typeCode: 9,
				   merchantId:117,
				   value: this.ordermeasurekName
	         }
           }
         }else if (name === 'addDictionary') {
          return {
            reqId: 0,
            channel: 0,
            os: "web",
            ver: "v2",
            appVer: "v2",
            model: "string",
            lng: "0",
            lat: "0",
            signType: "string",
            sign: "string",
            token: localStorage.getItem('token'),
            param: {
                   merchantId: 117,
                   value: this.shiftMetering,
                   typeCode:9,
                   status:0
                   
            }
          }
         }else if (name === 'editDictionary') {
          return {
            reqId: 0,
            channel: 0,
            os: "web",
            ver: "v2",
            appVer: "v2",
            model: "string",
            lng: "0",
            lat: "0",
            signType: "string",
            sign: "string",
            token: localStorage.getItem('token'),
            param: {
                   merchantId: 117,
                   value: this.shiftMetering,
                   typeCode:9,
                   id:this.meteringStatesid,
                   status:this.remarkState === true ? 0 : 1,
            }
          }
         }else if(name === 'deleteDictionary') {
	         	return {
			            reqId: 0,
			            channel: 0,
			            os: "web",
			            ver: "v2",
			            appVer: "v2",
			            model: "string",
			            lng: "0",
			            lat: "0",
			            signType: "string",
			            sign: "string",
			            token: localStorage.getItem('token'),
				          param:{
				        	   ids:this.ids
				    }   
	         }  
         }
      },
      
   },
  };
</script>

<style  lang="scss" rel="stylesheet/scss">
  .add {
    width: 88px;
    height: 36px;
    border-radius: 2px;
    background-color: #20a0ff;
    color: #fff;
    font-size: 14px;
    border: none;
    outline: none;
    cursor: pointer;
    &:hover {
      opacity: .9;
    }
  }

  .delAll {
    width: 88px;
    height: 36px;
    border-radius: 2px;
    background-color: #ffffff;
    border: solid 1px #c0ccda;
    font-size: 14px;
    color: #475669;
    cursor: pointer;
    &:hover {
      opacity: .9;
    }
  }
.ivu-table-header{
	width: 100%;
}
</style>