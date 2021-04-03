
<template style = "cursor:default" >

  <div id = "app"  >

    <div class = "jumbotron vertical-center" style="overflow-y: hidden; ">
        <div class="container rounded">
          <div class="row shadow p-3 mb-5 bg-white rounded" style="padding:12px !important ;margin-bottom:16px !important">
            <div class="col">
                <div class="d-flex justify-content-between">
                    
                        <button class="btn btn-danger btn-sm" v-on:click="pastday">◀</button>
                        <div style ="display: flex;align-items: center">
                          <input v-bind:class="[is_waiting == '1' ? 'waiting' : 'completed']" style="vertical-align:middle" type="date" id="date" name="date" v-model="date"   min="2018-01-01" max="2021-12-31">
                        </div>
                        
                        <button class="btn btn-danger btn-sm"  v-on:click="nextday">▶</button>
                    
                </div>
            </div>
          </div>

          <div class="row shadow p-3 mb-5 bg-white rounded" style="margin-bottom:16px !important">
                <div class="col" style="padding:0px;text-align:center; background-color:#EEEEEE">

                    {{ date2 }}

                </div>
          </div>

            <transition-group name="slide-fade" mode="out-in" tag="div" class="row shadow p-3 mb-5 bg-white rounded" style="margin-bottom:0px !important">
                <div  :key="meals[0]" v-for="(meals, mealtime) in meal_data" class="col-4 col table-responsive"  style = "border-left : 1px solid #E0E0E0; border-right : 1px solid #E0E0E0">
                    <table  v-bind:class="[time_status == mealtime && is_today == 1 ? 'active' : '']" style = "text-align : center;  width: 80%; margin: 0 auto !important;">
                        <tr style = "background-color : #EEE">
                            <th >
                                {{ mealtime }}
                            </th>
                        </tr>

                        <tr v-for="meal in meals" >

                                <td    style = "border-top : 1px solid #EEE;border-bottom : 1px solid #EEE; padding-top : 4px; padding-bottom :4px">
                                    {{ meal }}
                                </td>


                        </tr>

                    </table>
                </div>
            </transition-group>


        </div>
    </div>
  </div>
</template>

<!-- <body style="font-size:85%;min-width:550px;max-height:200px"> -->


<script>

//아침 : 6시 30분~ 8시 15분
//점심 : 8시 15분 : 13시 50분
//저녁 : 13시 50분 : 19시 50분 




import axios from 'axios';
import _ from 'lodash'


var today = new Date();


var hour = today.getHours()
var minute = today.getMinutes()

var time_status;

if ((hour == 6 && minute >= 30) || (hour == 7) || (hour == 8 && minute <= 15)) time_status = "아침"
else if ((hour == 8 && minute > 15) || (hour >= 9 && hour <= 12) || (hour == 13 && minute <= 50)) time_status = "점심"
else if ((hour == 13 && minute > 50) || (hour >= 14 && hour <= 18) || (hour == 19 && minute <= 50)) time_status = "저녁"


var dd = today.getDate();
var mm = today.getMonth()+1; //January is 0!
var yyyy = today.getFullYear();
if(dd<10) {
    dd='0'+dd
}
if(mm<10) {
    mm='0'+mm
}
var today_date = yyyy+"-"+mm+"-"+dd



export default {
  data() {
    return {
      meal_data : {"아침" : [], "점심" : [], "저녁" : []},
      date : today_date,
      date2 : "",
      time_status : time_status,
      is_waiting : 0,
      is_today : 1
    }
  },
  watch: {
    // 질문이 변경될 때 마다 이 기능이 실행됩니다.
    date: function (newDate) {
      this.is_waiting = 1;
      this.get_meal()
      
    }
  },
  methods: {
  	get_meal: _.debounce(
      function () {
             

             var date = this.date
             var vm = this

             axios.get(`https://api.dimigo.in/meal/date/${date}`)
              .then(function (response) {
                //alert(JSON.stringify(response.data));
                var res = response.data["meal"]
                console.log(res);
                var meal_data = {
                "아침": res["breakfast"] ? res["breakfast"]: ["급식 정보가 없습니다"],
                "점심":res["lunch"] ? res["lunch"] : ["급식 정보가 없습니다"],
                "저녁":res["dinner"] ? res["dinner"] : ["급식 정보가 없습니다"]
                }
                //alert(JSON.stringify(meal_data))
                
                vm.meal_data = meal_data
                
                var date2 = date.split("-").join("")
                //alert(date)
                var week = ['일', '월', '화', '수', '목', '금', '토'];
                var dayweek = week[new Date(vm.date).getDay()];
                

                date2 = date2.slice(0,4)+"년 "+date2.slice(4,6).replace(/(^0+)/, "")+"월 "+date2.slice(6,8).replace(/(^0+)/, "")+"일"+"("+dayweek+")"
          
                vm.date2 = date2
                console.log(date2)
                
                vm.is_waiting = 0
                
                
                var today = new Date()            
                var dd = today.getDate();
                var mm = today.getMonth()+1; //January is 0!
                var yyyy = today.getFullYear();
                if(dd<10) {
                    dd='0'+dd
                }
                if(mm<10) {
                    mm='0'+mm
                }
                var today_date = yyyy+"-"+mm+"-"+dd

                //alert(vm.date == today_date)
                if (vm.date == today_date){
                  vm.is_today = 1

                }else{
                  vm.is_today = 0

                }

                // var week = ['일', '월', '화', '수', '목', '금', '토'];
                // var dayweek = week[new Date(vm.date).getDay()];
                // vm.dayweek = dayweek;
                                
              })
              .catch(function (error) {
                //alert("ASDF")
                var meal_data = {"아침":["급식 정보가 없습니다"], "점심":["급식 정보가 없습니다"], "저녁":["급식 정보가 없습니다"]}
                vm.meal_data = meal_data
                var date2 = date.split("-").join("")
                //alert(date)


                var week = ['일', '월', '화', '수', '목', '금', '토'];
                var dayweek = week[new Date(vm.date).getDay()];


                date2 = date2.slice(0,4)+"년 "+date2.slice(4,6).replace(/(^0+)/, "")+"월 "+date2.slice(6,8).replace(/(^0+)/, "")+"일"+"("+dayweek+")"
                vm.date2 = date2
                vm.is_waiting = 0

                var today = new Date()            
                var dd = today.getDate();
                var mm = today.getMonth()+1; //January is 0!
                var yyyy = today.getFullYear();
                if(dd<10) {
                    dd='0'+dd
                }
                if(mm<10) {
                    mm='0'+mm
                }
                var today_date = yyyy+"-"+mm+"-"+dd

                //alert(vm.date == today_date)
                if (vm.date == today_date){
                  vm.is_today = 1

                }else{
                  vm.is_today = 0

                }

                // var week = ['일', '월', '화', '수', '목', '금', '토'];
                // var dayweek = week[new Date(vm.date).getDay()];
                // vm.dayweek = dayweek;
                
              })

               
        },
        250
    ),
        nextday:
          function () {

                var date = new Date(document.getElementById("date").value);
                date.setDate(date.getDate() + parseInt(1));
                date = date.toISOString().substring(0, 10);
                document.getElementById("date").value = date;
                this.date = date

          },

        pastday:
          function () {

                var date = new Date(document.getElementById("date").value);
                date.setDate(date.getDate() + parseInt(-1));
                date = date.toISOString().substring(0, 10);
                document.getElementById("date").value = date;
                this.date = date
          }
        


        //this .meal_data = {"아침" : ["아침1", "아침12321232"], "점심" : meal_lunch, "저녁" : meal_dinner}
  },
  created (){
    this.get_meal()
  }
  
};


</script>


<style scoped>

@import url('https://fonts.googleapis.com/css?family=Nanum+Gothic&display=swap');

::-webkit-scrollbar {width: 0 !important; height: 0 !important}



html, body {
  height: 100%;
  overflow:hidden;
}


table {
  user-select : none
}

table th {
  border-radius: 5px;
} 




table.active th{
 color : white;
 background-color : #EF5350; 
}


table.active td{
  background-color:rgb(255, 244, 244);
  color : black;
  font-weight : 550;
  border-bottom: rgb(255, 211, 211) 1px solid !important;
}



[type="date"] {
  background:#fff url(https://cdn1.iconfinder.com/data/icons/cc_mono_icon_set/blacks/16x16/calendar_2.png)  97% 50% no-repeat ;
}
input[type=date]::-webkit-inner-spin-button {
    -webkit-appearance: none;
    display: none;
}
[type="date"]::-webkit-calendar-picker-indicator {
  opacity: 0;
}
::-webkit-clear-button
{
    display: none; /* Hide the button */
    -webkit-appearance: none; /* turn off default browser styling */
}



::-webkit-scrollbar {
    width: 0px;  /* Remove scrollbar space */
    background: transparent;  /* Optional: just make scrollbar invisible */
}
/* Optional: show position indicator in red */




/* custom styles */
body {
  padding: 4em;
  background: #e5e5e5;
  font: 13px/1.4 Geneva, 'Lucida Sans', 'Lucida Grande', 'Lucida Sans Unicode', Verdana, sans-serif;
}
label {
  display: block;
}
input {
  border: 1px solid #c4c4c4;
  border-radius: 15px;
 
  padding: 3px 5px;
  box-shadow:  0 3px 6px rgba(0,0,0,0.1);
  font-size:14px;
  width: 190px;
}

input.completed{
  background-color: white;
}

input.waiting {

  background-color: #eee ;

}

input[type=date]{
  text-align:center
}




.element::-webkit-scrollbar { width: 0 !important }

.btn-danger {
  background-color : rgb(237,18,128);
  box-shadow:  0 3px 6px rgba(0,0,0,0.1);
}

.jumbotron.vertical-center, .vertical-center {
    margin-bottom: 0; /* Remove the default bottom margin of .jumbotron */
    background-color: white;
    background-image: linear-gradient(45deg, #EEE 25%, transparent 25%, transparent 75%, #EEE 75%, #EEE),
    linear-gradient(45deg, #EEE 25%, transparent 25%, transparent 75%, #EEE 75%, #EEE);
    background-size: 180px 180px;
    background-position: 0 0, 90px 90px;
}
/*
.vertical-center {
  min-height: 100%;  
  min-height: 100vh; 

  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;

    -webkit-box-align : center;
  -webkit-align-items : center;
       -moz-box-align : center;
       -ms-flex-align : center;
          align-items : center;

  width: 100%;


         -webkit-box-pack : center;
            -moz-box-pack : center;
            -ms-flex-pack : center;
  -webkit-justify-content : center;
          justify-content : center;
}
*/

.container {
  background-color: rgb(237,18,128);
}

#nprogress .bar {
  background: #ed1280 !important;
}
#nprogress .spinner-icon {
  display:none;
}



.slide-fade-enter-active {
    transition : all .5s;
}

.slide-fade-leave-active {
  transition: all 0s;

}


.slide-fade-enter, .slide-fade-leave-to /* .list-leave-active below version 2.1.8 */ {
  opacity: 0;

  transform: perspective(500px) rotate3d(0, 1, 0, 180deg);

}





</style>