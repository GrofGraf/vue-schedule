<template>
<div>
<table>
  <thead>
    <tr>
      <th>Room ID</th>
      <th v-for="day in neededDates" :class="{'sunday': day.format('d')==0, 'today': day==today}">
        {{day.format("dd")}}
        <br/>
        {{day.format("D")}}
      </th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="room in rooms" :id="'room-'+room.id">
      <th>
        {{room.id}}
      </th>
      <td v-for="(day, index) in neededDates" :date="day.format('YYYY-MM-DD')" :room-id="room.id" v-on:click="tdClicked($event)" :tabindex="hasEvent(day, room.id) ? null : 1">
        <div class="event" v-for="event in hasEvent(day, room.id)" :id="'event-'+event.id" v-on:click="eventClicked($event)" :class="{'event-start': event.startAt==day.format('YYYY-MM-DD'), 'event-end': event.endAt==day.format('YYYY-MM-DD')}" :title="event.title">
          <div v-if="event.startAt==day.format('YYYY-MM-DD')||index==0" class="avatar-container">
            <img :src="'./avatars/'+event.avatar" class="avatar">
          </div>
        </div>
      </td>
    </tr>
  </tbody>
</table>
<div v-if="!!newEvent">
	<input v-model="newEvent.startAt">
	<input v-model="newEvent.endAt">
	<button v-on:click="cancel()">Cancel</button>
	<button v-on:click="saveEvent()">Save</button>
</div>
</div>
</template>

<script>
import moment from 'moment'

var events = [{id:1, startAt: "2016-12-11", endAt: "2016-12-24", roomId: "3", title: "Airbnb Reservation", avatar:"airbnb.png"}, {id:2, startAt: "2016-12-11", endAt: "2016-12-13", roomId: "2", title: "Booking reservation", avatar: "booking.png"}, {id:3, startAt: "2016-12-15", endAt: "2016-12-16", roomId: "2", title: "Agoda reservation", avatar: "agoda.png"}, {id:4, startAt: "2016-12-16", endAt: "2016-12-18", roomId: "2", title: "Tripadvisor reservation", avatar: "tripadvisor.png"}]
var rooms = [{id:1},{id:2},{id:3}]
var date=moment();
var view="week";
var weekStart = "1";

export default {
  data () {
    return {
      events: events,
			today: moment(),
			date: date,
			rooms: rooms,
			view: view,
			weekStart: weekStart,
			dates: [],
			newEvent:"",
    }
  },
  computed: {
		dateStart(){
			return moment(this.date).startOf(this.view).add(this.weekStart, "days");
		},
		dateEnd(){
			return moment(this.date).endOf(this.view).add(this.weekStart, "days");
		},
		neededDates(){
			this.dates=[];
			var start=this.dateStart;
			while (start <= this.dateEnd) {
				this.dates.push(start);
				start = start.clone().add(1, 'days');
			}
			return this.dates;
		},
		neededEvents(){
			//za vsak teden/mesec rabim evente, ki imajo startAt<weekEnd && endAt>weekStart
			var start = this.dateStart.format('YYYY-MM-DD');
			var end = this.dateEnd.format('YYYY-MM-DD')
			var events=this.events.filter(function(e){
				return e.startAt<=end && e.endAt>=start;
			})
			return events;
		}
	},
	methods: {
		hasEvent(day, room){
			var dayEvents = [];
			for(var i=0, len=this.neededEvents.length; i<len; i++){
				this.neededEvents[i].roomId==room && this.neededEvents[i].startAt <= day.format("YYYY-MM-DD") && this.neededEvents[i].endAt >= day.format("YYYY-MM-DD") && dayEvents.push(this.neededEvents[i]);
			}
			if(dayEvents.length > 0){
				return dayEvents;
			};
			return false;
		},
		nextDates(){
			this.date=this.date.clone().add(1, this.view);
			console.log("tukituki")
		},
		previousDates(){
			this.date=this.date.clone().subtract(1, this.view);
		},
		tdClicked(ev){
			var roomId = ev.currentTarget.room-id;
			var date = ev.currentTarget.date;
			console.log(roomId, date)
			var clickedBox = document.querySelector("td[room-id='" + roomId + "'][date='" +date.format("YYYY-MM-DD")+ "']");
			console.log(clickedBox)
			if(clickedBox.hasChild){
				console.log("lalalala")
				return;
			}else{
				this.newEvent = {id:5, startAt:date.format("YYYY-MM-DD"), title: "LazyHotelier", avatar: "lazy.png", roomId: roomId};
				//clickedBox.focus();
			}
		},
		eventClicked(ev){
			console.log(ev)
			var clickedEvent=this.neededEvents.find(function(e){
				return "event-"+e.id==ev.currentTarget.id;
			})
			console.log(clickedEvent.title);
		},
		saveEvent(){
			this.events.push(this.newEvent);
			this.newEvent = "";
		},
		cancel(){
			this.newEvent="";
		}
	}
}
</script>

<style>
table{
	border-collapse: collapse;
	width:100%;
	overflow: hidden;
}
tbody{
	position: relative;
}
tbody tr:nth-child(odd) {
   background-color: #dce0e0;
}
tr{
	position:relative;
	height:50px;
}
th{
	border: solid 1px;
}
td{
	border: solid 1px;
	position: relative;
}
td:focus {
	background-color: aqua;
}
.sunday{
	background-color: #ff5a5f;
}
.event{
	position: absolute;
	left:-1px;
	height: 15px;
	width: calc(100% + 2px);
	background-color:#00a699;
	z-index: 2;
}
.event-start{
	left: 30%;
	width: calc(70% + 1px);
	border-top-left-radius: 25px;
	border-bottom-left-radius: 25px;
	z-index: 3;
}
.event-end{
	width: calc(10% + 1px);
	right: 90%;
	border-top-right-radius: 25px;
	border-bottom-right-radius: 25px;
	z-index: 1;
}
.avatar-container{
	position: absolute;
	bottom: -50%;
	left: 20%;
	height:40px;
	width: 40px;
	border: solid 2px #00a699;
	border-radius: 25px;
	background-color: white;
	overflow:hidden;
}
.avatar{
	height: 100%;
	width: 100%;
}
</style>
