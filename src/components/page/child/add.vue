<template>
<div>
	<mt-header fixed  title="宝宝信息">
	<router-link :to="{name:'home'}" slot="left">
	    <mt-button class="el-icon-arrow-left"></mt-button>
	</router-link>
	</mt-header>
	<div class="page-header-main pdb-50">
		<div class="text-center pdt-10" v-on:click="pickerImage()">
			<div class="headPic-big">
				<div class="img-bg" :style="{backgroundImage:'url('+headPic+')'}"></div>
			</div>
			<span class="color-desc fontsize-small">请点击上传宝宝头像</span>
			<input style="display:none;" id="fileInput" type="file" v-on:change="selectImage()" accept="image/*"/>
		</div>
		<div class="clipperVisibleBox" v-show="clipperVisible">
			<div class="clipperImage"><img :src="clipperImg" id="headPic"/></div>
			<div class="clipperTool">
				<mt-button size="small"  type="primary" v-on:click="cropperRotate(-45)" class="mrr-10">
					<span class="fa fa-rotate-left"></span>
				</mt-button>
				<mt-button size="small"  type="primary" v-on:click="cropperRotate(45)" class="mrr-10">
					<span class="fa fa-rotate-right"></span>
				</mt-button>
				<mt-button size="small"  class="right" v-on:click="cancelCropperCanvasData">
					取消
				</mt-button>
				<mt-button size="small"  type="primary" class="mrr-10 right" v-on:click="getCropperCanvasData">
					确定
				</mt-button>
				
			</div>
		</div>
		<div class="mint-cells-form mrt-20">
			<div class="mint-cell">
				<label class="mint-cell-form-label">昵称:</label>
				<div class="mint-cell-primary">
					<input type="text" v-validate="'required'" :class="{'minit-cell-form-input mrt-10':true,'is-error':errors.has('nickName')}" name="nickName" v-model="nickName" maxlength="4" placeholder="请输入宝宝的昵称"/>
					<div v-show="errors.has('nickName')" class="validate-error">宝宝的昵称必须填写</div>
				</div>
			</div>
			<div class="mint-cell">
				<label class="mint-cell-form-label">生日:</label>
				<div class="mint-cell-primary">
					<input type="text" onfocus="this.blur()" class="minit-cell-form-input mrt-10" v-model="birthDate" placeholder="请输入宝宝的生日" v-on:click="openPicker" readonly/>
				</div>
			</div>
			<div class="mint-cell">
				<label class="mint-cell-form-label">性别:</label>
				<div class="mint-cell-primary">
					<mt-radio class="sex-radio" 
					  v-model="sex"
					  :options="sexOptions">
					</mt-radio>
				</div>
			</div>
		</div>
		<div class="text-center mrt-50 mrl-10 mrr-10">
			<mt-button class="mint-button--full" type="primary" v-on:click="saveData">保存</mt-button>
		</div>
	</div>
<mt-datetime-picker
ref="picker" 
v-model="pickerValue"
type="date" 
@confirm="handleDateConfirm" 
@cancel="handleDateCancel"
>
</mt-datetime-picker>
</div>
</template>

<script>
	import cropper from 'cropperjs';
	let _cropper;
	export default { 
		name: 'child-add',
		data(){
			return {
				id:-1,
				clipperVisible:false,
				pickerValue:new Date().format('yyyy-MM-dd'),
				sexOptions:[
				  {
				    label: '男',
				    value: '0'
				  },
				  {
				    label: '女',
				    value: '1'
				  }
				],
				nickName:'',
				birthDate:new Date().format('yyyy-MM-dd'),
				sex:'0',
				headPic:require('../../../images/child.png'),
				clipperImg:require('../../../images/child.png')
			}
		},
		created(){
			let _this = this;
			let _childList = _this.$localData.getItemArray(_this.$config.localStorageKey.childData);
			_this.id = _this.$route.query.id;
			if(!_childList[_this.id]){
				_this.id = -1;
			}else{
				_this.nickName = _childList[_this.id].nickName;
				_this.birthDate = _childList[_this.id].birthDate;
				_this.headPic = _childList[_this.id].headPic;
				_this.sex = _childList[_this.id].sex;
			}
		},
		methods: {
			pickerImage(){
				document.querySelector('#fileInput').click();
			},
			cropperRotate(rotate){
				_cropper.rotate(rotate);
			},
			cancelCropperCanvasData(){
				let _this = this;
				_this.clipperVisible = false;
				_cropper.destroy();
			},
			getCropperCanvasData(){
				let _this = this;
				_this.headPic = _cropper.getCroppedCanvas({fillColor:'#fff'}).toDataURL('image/jpeg');
				_this.clipperVisible = false;
				_cropper.destroy();
			},
			selectImage(){
				let _this = this,
					_fileSelector = document.querySelector('#fileInput'),
					_file = _fileSelector.files[0],
					_reader = new FileReader();
				if(!_file){
					return;
				}
			    _reader.readAsDataURL(_file);
			    _this.$indicator.open();
			    _reader.onload = function(e){
			        _this.clipperImg = e.target.result;
			        _this.clipperVisible = true;
			        
					setTimeout(function(){
						let _image = document.getElementById('headPic');
						_cropper = new cropper(_image, {
							aspectRatio:1/1,
							zoomOnWheel:false,
						});
						_this.$indicator.close();
					},200);
			    };
			},
			handleDateCancel(){
				document.body.classList.remove('scrollY-hiden');
			},
			handleDateConfirm(d){
				document.body.classList.remove('scrollY-hiden');
				this.birthDate = d.format('yyyy-MM-dd');
				this.$refs.picker.close();
			},
			openPicker() {
				document.body.classList.add('scrollY-hiden');
		    	this.$refs.picker.open();
		    },
		    saveData(){
		    	let _this = this;
		    	let _childList = [];
				_this.$validator.validateAll().then(result => {
					if(result){
						_childList = _this.$localData.getItemArray(_this.$config.localStorageKey.childData);
						if(_this.id==-1){
					    	_childList.push({
					    		headPic:_this.headPic,
					    		nickName:_this.nickName,
					    		sex:_this.sex,
					    		birthDate:_this.birthDate,
					    		weightList:[],
					    		heightList:[]
					    	});
				    	}else{
				    		_childList[_this.id].headPic = _this.headPic;
				    		_childList[_this.id].nickName = _this.nickName;
				    		_childList[_this.id].sex = _this.sex;
				    		_childList[_this.id].birthDate = _this.birthDate;
				    	}
				    	_this.$localData.setItem(_this.$config.localStorageKey.childData,_childList);
				    	this.$router.push({
							name:'home' 
						});
					}
				});
		    }
	  	}
	}
</script>