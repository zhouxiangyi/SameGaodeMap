<template>
  <div class="cityChange">
      <ul>
        <li class="citychose">
              <el-dropdown type="primary" trigger="click">
                  <span class="el-dropdown-link">
                  {{city}}
                  <i class="el-icon-arrow-down el-icon--right"></i>
                  </span>
                   <el-dropdown-menu slot="dropdown">
                       <city-chose-list @currentcity="getcurrentcity" :currentcitys="city"></city-chose-list>
                  </el-dropdown-menu>
              </el-dropdown>
        </li>
        <li class="weather">
            <span class="weature">
                {{weature}} /
            </span>
            <span class="tempture">
                {{temperature}}℃
            </span>
        </li>
     </ul>
  </div>
</template>

<script>
import { transform } from 'ol/proj'
import {mapconfig} from '../assets/config/mapconfig'
import CityChoseList from '@/components/citychoselist.vue'
export default {
    name:'CityChose',
    data(){
        return{
            center:'',
            city:'',
            cityadccode:'',
            weature:'',
            temperature:''
        }
    },
    watch:{
        cityadccode(){
            //天气查询
            this.GetWeature()
        }
    },
    computed:{
       map:{
           get(){
                 return this.$store.state.map
           }
          
       }
    },
    mounted(){
        //设置延时确保可以获取到map
        setTimeout(()=>{
             console.log(this.map)
             const center = transform(this.map.getView().getCenter(), 'EPSG:3857', 'EPSG:4326')
             this.center = center.join(",");
             this.InverseGeo()
        },800)
       
        //this.center = transform(this.map.getView().getCenter(), 'EPSG:3857', 'EPSG:4326')
      
    },
    methods:{
        //逆地址解析获取当前城市
        InverseGeo(){
            const url = mapconfig.InverseGeoUrl
            this.$axios.get(url,{
                params:{
                    key:mapconfig.AmapServeKey,
                    location:this.center
                }
            }).then((res)=>{
                console.log(res)
                if(res.statusText==='OK'){
                    const data = res.data
                    //城市的名字
                    this.city = data.regeocode.addressComponent.city
                    //城市的编码
                    this.cityadccode =  data.regeocode.addressComponent.adcode
                    
                }
            })
        },
        //天气查询
        GetWeature(){
            this.$axios.get(mapconfig.GetWeatureUrl,{
                params:{
                    key:mapconfig.AmapServeKey,
                    city:this.cityadccode
                }
            }).then((res)=>{
                if(res.statusText==="OK"){
                    const data = res.data.lives[0]
                    this.weature = data.weather
                    this.temperature =data.temperature
                }
            })
        },
        //正地址解析
        Getadacode(cityname){
            const url = mapconfig.Geourl
            this.$axios.get(url,{
                params:{
                    key:mapconfig.AmapServeKey,
                    address:cityname
                }
            }).then((res)=>{
                console.log(res)
                if(res.statusText==='OK'){
                    const data = res.data
                    //城市的编码
                    this.cityadccode =  data.geocodes[0].adcode
                    
                }
            })
        },
        getcurrentcity(cityname){
            console.log(cityname)
            this.city = cityname
            if(cityname==="全国"){
                this.weature = ''
                this.temperature = ''
                return
            }
            //正地址解析获取code
            this.Getadacode(cityname)
        }
    },
    components:{
        CityChoseList
    }

}
</script>


<style lang="stylus" scoped>
@import '~styles/varibles.styl'
@import '~styles/mixins.styl'
@import '~styles/common.css'
.cityChange
  defaultdiv()
  top: 0.32rem;
  left: 8rem;
  font-size: 0.28rem;
  background: #fff;
  border:none;
  cursor:pointer
  ul
    margin:0
    padding:0
    display: flex;
    flex-direction: row;
    li
        display:inline-block
        padding:.16rem
    .citychose   
        font-weight:bold 
    .citychose:hover
        color:blue
        i
            font-weight:bold 
    .weather
        padding-left: 0;
</style>