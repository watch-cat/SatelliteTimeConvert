<template>
    <div class="time-frame">
        <h3>协调世界时（UTC）</h3>
        <span>请输入需要转换的世界时：</span>
        <input type="date" min="1980-01-06" v-model="selectedDate">
        <div>
            输入的日期为：
            <input class="number-input" type="number" min="1980" v-model="utcYear">
            年第
            <input class="number-input" type="number" min="1" max="366" v-model="utcDay">
            天
        </div>
    </div>

    <div class="time-frame">
        <h3>GPS时间系统</h3>
        <div>
            对应的GPS时间为：第
            <input class="number-input" type="number" min="0" v-model="gpsWeek">
<!--            <span class="result-text">{{gpsWeek}}</span>-->
            周第
            <input class="number-input" type="number" min="0" max="6" v-model="gpsDay">
<!--            <span class="result-text">{{gpsDay}}</span>-->
            天
        </div>
    </div>

    <div class="text-frame">
        <p>注1：UTC时间的天数以1为起始，GPS时间的周数和天数以0为起始</p>
        <p>注2：得益于响应式设计，输入需要转换的时间后，无需点击任何按钮，时间将自动转换</p>
    </div>

</template>

<script>
import {onMounted, ref, watch} from "vue";

export default {
    name: "UTC2GPSTime",
    setup(){
        onMounted(() => {
            setCurrentDate()
        })

        const selectedDate = ref('')

        const utcYear = ref(0);
        const utcDay = ref(0);

        const gpsWeek = ref(0);
        const gpsDay = ref(0);

        watch(selectedDate, (newDate) => {
            if (newDate){
                const [year, month, day] = newDate.split('-').map(Number)
                utcYear.value = year
                utcDay.value = getDayOfYear(year, month, day)
                gpsWeek.value = getGPSWeekDay(year, month, day).weekOfGPS
                gpsDay.value = getGPSWeekDay(year, month, day).dayOfGPS2
            }
        })

        watch([utcYear, utcDay], ([newUtcYear, newUtcDay]) => {
            const msInOneDay = 1000*60*60*24
            const unixStartDate = new Date(Date.UTC(1970, 0, 1))
            const yearStartDate = new Date(Date.UTC(newUtcYear, 0, 1))

            if (newUtcYear > 1980 && newUtcDay > 1){
                const newDate = new Date(yearStartDate - unixStartDate + (newUtcDay - 1) * msInOneDay)
                selectedDate.value = newDate.toISOString().slice(0, 10)
            }
        })

        watch([gpsWeek, gpsDay], ([newGpsWeek, newGpsDay]) => {
            const msInOneWeek = 1000*60*60*24*7
            const msInOneDay = 1000*60*60*24
            const gpsStartDate = new Date(Date.UTC(1980, 0, 6))
            const unixStartDate = new Date(Date.UTC(1970, 0, 1))

            const newDate = new Date(gpsStartDate - unixStartDate + newGpsWeek * msInOneWeek + newGpsDay * msInOneDay)

            selectedDate.value = newDate.toISOString().slice(0, 10)
            //console.log(gpsStartDate + Math.trunc(newGpsWeek * msInOneWeek + newGpsDay * msInOneDay) + ' ' + newGpsWeek + newGpsDay)
        })

        function setCurrentDate(){
            const currentDate = new Date()
            selectedDate.value = currentDate.toISOString().slice(0, 10)
        }

        function getDayOfYear(year, month, day){
            const inputDate = new Date(Date.UTC(year, month - 1, day))
            const startDate = new Date(Date.UTC(year, 0, 1))

            const dayOfYear = Math.trunc((inputDate - startDate)/(1000*60*60*24) + 1)
            return dayOfYear
        }

        function getGPSWeekDay(year, month, day){
            const inputDate = new Date(Date.UTC(year, month - 1, day))
            const gpsStartDate = new Date(Date.UTC(1980, 0, 6))

            const msInOneWeek = 1000*60*60*24*7
            const msInOneDay = 1000*60*60*24
            const weekOfGPS = Math.trunc((inputDate - gpsStartDate)/msInOneWeek)
            const dayOfGPS2 = (inputDate - gpsStartDate - weekOfGPS*msInOneWeek)/msInOneDay
            const dayOfGPS = Math.trunc((inputDate - gpsStartDate - weekOfGPS*msInOneWeek)/msInOneDay)
            return {weekOfGPS, dayOfGPS2};
        }

        return{
            selectedDate,
            utcYear,
            utcDay,
            gpsWeek,
            gpsDay
        }
    }
}
</script>

<style scoped>
.time-frame {
    padding: 0 20px 20px 20px;
    margin: 20px 0;
    border: 1px solid #e0e0e0;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
}

.text-frame {
    padding: 10px;
    margin: 20px 0;
    font-size: 14px;
    color: #555555;
}

.result-text{
    font-family: 'Comic Sans MS', cursive;
    font-weight: bold;
    font-size: 18px;
}

.number-input{
    width: 60px;
}

h3 {
    font-size: 20px;
    color: #333333;
    margin-bottom: 15px;
}

/*span {*/
/*    font-size: 16px;*/
/*    margin-right: 10px;*/
/*    color: #666666;*/
/*}*/

input {
    padding: 8px;
    font-size: 16px;
    border: 1px solid #cccccc;
    border-radius: 4px;
}

div {
    margin-top: 10px;
    font-size: 16px;
    color: #555555;
}
</style>