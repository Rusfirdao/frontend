<template>
  <q-page class="q-pa-sm">
    <section-header
      title="Report"
      subTitle="แสดงข้อมูลการใช้งานของ Users"
    ></section-header>
    <q-tabs v-model="tab" class="q-mb-lg">
      <q-tab class="text-purple" name="Dairy" label="Dairy" />
      <q-tab class="text-orange" name="Monthly" label="Monthly" />
      <q-tab class="text-teal" name="Select Date" label="Select Date" />
    </q-tabs>

    <q-tab-panels
      v-model="tab"
      animated
      transition-prev="fade"
      transition-next="fade"
      style="background-color: #eceff1"
    >
      <!-- =================================================== Print Day ==================================================================== -->
      <q-tab-panel name="Dairy">
        <div class="q-pa-md">
          <q-table
            :title="`Timeline Dairy - ` + this.date"
            :data="list_day"
            :columns="columns"
            row-key="name"
            class="rounded-borders-10 shadow-5"
          >
            <template v-slot:header="props">
              <q-tr :props="props">
                <q-th auto-width />
                <q-th v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.label }}
                </q-th>
              </q-tr>
            </template>

            <template v-slot:body="props">
              <q-tr :props="props">
                <q-td auto-width>
                  <q-btn
                    size="md"
                    color="primary-gradient"
                    round
                    :to="'/timeline/' + props.row.id"
                    :icon="props.expand ? '' : 'person_pin_circle'"
                  />
                </q-td>
                <q-td v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.value }}
                </q-td>
              </q-tr>
            </template>
          </q-table>
        </div>
      </q-tab-panel>
      <!-- =========================================== Print Mounthly =========================================================== -->
      <q-tab-panel name="Monthly">
        <div class="q-pa-md">
          <q-table
            :title="`Timeline Monthly - ` + this.month"
            :data="list_month"
            :columns="columns"
            row-key="name"
            class="rounded-borders-10 shadow-5"
          >
            <template v-slot:header="props">
              <q-tr :props="props">
                <q-th auto-width />
                <q-th v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.label }}
                </q-th>
              </q-tr>
            </template>

            <template v-slot:body="props">
              <q-tr :props="props">
                <q-td auto-width>
                  <q-btn
                    size="md"
                    color="primary-gradient"
                    round
                    :to="'/timeline/' + props.row.id"
                    :icon="props.expand ? '' : 'person_pin_circle'"
                  />
                </q-td>
                <q-td v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.value }}
                </q-td>
              </q-tr>
            </template>
          </q-table>
        </div>
      </q-tab-panel>
      <!-- ================================================ Print Select Date =============================================================== -->
      <q-tab-panel name="Select Date">
        <div class="col">
          <div class="row-4">Select Date</div>
          <div class="row">
            <div class="q-pa-md col self-center" style="max-width: 300px">
            <q-input filled v-model="date" mask="date" :rules="['date']" label="Select Date">
              <template v-slot:append>
                <q-icon name="event" class="cursor-pointer">
                  <q-popup-proxy
                    ref="qDateProxy"
                    transition-show="scale"
                    transition-hide="scale"
                  >
                    <q-date v-model="date">
                      <div class="row items-center justify-end">
                        <q-btn
                          v-close-popup
                          label="Select"
                          color="primary" 
                          flat
                        />
                      </div>
                    </q-date>
                  </q-popup-proxy>
                </q-icon>
              </template>
            </q-input>
          </div>
          <div class="q-pa-md col" style="max-width: 300px">
          <q-select filled v-model="select_room" :options="options" label="Select Room" />
          </div>
          <div class="q-gutter-md  self-center" style="max-width: 300px">
            <q-btn
             icon="check"
             color="primary-gradient"
             label="Submit" type="submit"
             @click="select()"
             clickable
            />
            <q-btn
             color="primary-gradient"
             label="Show all data" type="submit"
             @click="showdata()"
             clickable
            />
          </div>
          
          </div>
        </div>

        <div class="q-pa-md">
          <q-table
            title="Timeline"
            :data="list_select"
            :columns="columns"
            row-key="name"
            class="rounded-borders-10 shadow-5"
          >
            <template v-slot:header="props">
              <q-tr :props="props">
                <q-th auto-width />
                <q-th v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.label }}
                </q-th>
              </q-tr>
            </template>
            <template v-slot:body="props">
              <q-tr :props="props">
                <q-td auto-width>
                  <q-btn
                    size="md"
                    color="primary-gradient"
                    round
                    :id="0"
                    :to="'/timeline/' + props.row.id"
                    :icon="props.expand ? '' : 'person_pin_circle'"
                  />
                </q-td>
                <q-td v-for="col in props.cols" :key="col.name" :props="props">
                  {{ col.value }}
                </q-td>
              </q-tr>
            </template>
          </q-table>
        </div>
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script>
import SectionHeader from "../components/SectionHeader.vue";
const moment = require("moment");
import { axios } from "boot/axios";
import { ref } from 'vue'
export default {
  components: {
    SectionHeader,
  },
  data() {
    return {
      tab: "Dairy",
      select_room: null,
      date: null,
      month: moment().format("YYYY/MM"),
      list_month: [],
      list_day: [],
      list_select: [],
      dashbord: [],
      options: [],
      columns: [
        {
          name: "name",
          required: true,
          label: "Name",
          align: "left",
          field: (row) => row.name,
          format: (val) => `${val}`,
          sortable: true,
        },
        {
          name: "date",
          align: "center",
          label: "Date",
          field: "date",
          sortable: true,
        },
        {
          name: "time-start",
          align: "center",
          label: "Time Start",
          field: "time_start",
          sortable: true,
        },
        {
          name: "time-stop",
          align: "center",
          label: "Time Stop",
          field: "time_stop",
          sortable: true,
        },
        {
          name: "type",
          align: "center",
          label: "Type",
          field: "type",
          sortable: true,
        },
        {
          name: "location",
          align: "center",
          label: "Location",
          field: "location",
          sortable: true,
        },
      ],
    };
      
  },
  async mounted() {
    
    // setTimeout(function () {
    //   location.reload(1);
    // }, 60000);
    //<------------------------- Connect Database ----------------------------------->
    const url = "http://localhost:3030/api/" 
    let resp1 = await axios.get(url+"visitors");
    this.list1 = resp1.data.result.rows;
    console.warn("Visitor ");
    console.warn(resp1.data.result.rows);
    let resp2 = await axios.get(url+"scanlog");
    this.list2 = resp2.data.result.rows;
    console.warn("list2 scanerlog kk");
    console.warn(this.list2);
    let resp3 = await axios.get(url+"locations");
    this.list3 = resp3.data.result.rows;
    console.warn("list3 room");
    console.warn(this.list3);
    for(var k = 0; k < this.list3.length; k++){
      this.options.push(this.list3[k].room)
    }
    console.warn(this.options)
  //<------------------------- Create Dashbord ----------------------------------->
    console.warn("Time -------------- : "+moment().format())
    console.warn("Time -------------- : "+moment().format("hh:mm A"))
    for (var i = 0; i < this.list1.length; i++) {
      for (var j = 0; j < this.list2.length; j++) {
      // console.warn("visitor_id l : "+this.list2[j].scan_timestamp)
      moment().format("hh:mm A")
        // console.warn("Test login "+ i+" : " +moment(this.list1[i].time_stop).format("hh:mm A")+" : "+ j +" : "+moment(this.list2[j].scan_timestamp).format("hh:mm A"))
        if (this.list1[i].time_stop == null) {
          // console.warn("visitor_id In : "+this.list1[i].visitor_id)
          const newItem = {
            id: this.list1[i].visitor_id,
            date: moment(this.list1[i].time_start).format("YYYY-MM-DD"),
            location: this.list2[j].room,
            name: this.list1[i].first_name + " " + this.list1[i].last_name,
            time_start: moment(this.list1[i].time_start).format("hh:mm"),
            time_stop: "In Use",
            type: this.list1[i].category,
            tag_address:this.list1[i].tag_address
          };
          this.dashbord.push(newItem);
          break
        }else if(moment(this.list1[i].time_stop).format("hh:mm A") == moment(this.list2[j].scan_timestamp).format("hh:mm A")){
          // console.warn("visitor_id Out : "+this.list1[i].visitor_id)
          const newItem = {
            id: this.list1[i].visitor_id,
            date: moment(this.list1[i].time_start).format("YYYY-MM-DD"),
            location: this.list2[j].room,
            name: this.list1[i].first_name + " " + this.list1[i].last_name,
            time_start: moment(this.list1[i].time_start).format("hh:mm"),
            time_stop: moment(this.list1[i].time_stop).format("hh:mm"),
            type: this.list1[i].category,
            tag_address:this.list1[i].tag_address
          };
          this.dashbord.push(newItem);
          break
        }
      }
    }
    console.warn("Time line all")
    console.warn(this.dashbord)
      for (var i = 0; i < this.dashbord.length; i++) {
        
      }
      // console.warn("Time line month //")
      // console.warn(this.list_month);
      for (var i = 0; i < this.dashbord.length; i++) {
        //<------------------------- List Day ----------------------------------->
        if (
          moment(this.dashbord[i].date).format("YYYY-MM-DD") ==
          moment().format("YYYY-MM-DD")
        ) {
          const newItems = {
            id: this.dashbord[i].id,
            date: this.dashbord[i].date,
            location: this.dashbord[i].location,
            name: this.dashbord[i].name,
            time_start: this.dashbord[i].time_start,
            time_stop: this.dashbord[i].time_stop,
            type: this.dashbord[i].type,
          };
          this.list_day.push(newItems);
        }
        //<------------------------- List Mounth ----------------------------------->
        if (
          moment(this.dashbord[i].date).format("YYYY-MM") ==
          moment().format("YYYY-MM")
        ) {
          const newItems = {
            id: this.dashbord[i].id,
            date: this.dashbord[i].date,
            location: this.dashbord[i].location,
            name: this.dashbord[i].name,
            time_start: this.dashbord[i].time_start,
            time_stop: this.dashbord[i].time_stop,
            type: this.dashbord[i].type,
          };
          this.list_month.push(newItems);
        }
      }
      //<------------------------- Print Test ----------------------------------->
      console.warn("List Day ")
      console.warn(this.list_day);
      console.warn("List Mounth ")
      console.warn(this.list_month);
      this.list_select = this.dashbord;
      console.warn(this.list_select);
    },
    methods: {
      //<------------------------- Select Date ----------------------------------->
      async select() {
        // console.warn(this.date);
        console.warn("Date : "+this.date)
        console.warn("Room : "+this.select_room)
         this.list_select = [];
        if(this.date==null&&this.select_room==null){
          this.list_select = this.dashbord;
          console.warn("Test : 1 ==")
        }else if(this.date!=null&&this.select_room==null){
          console.warn("Test : 2 !=")
          for (var i = 0; i < this.dashbord.length; i++) {
          if (moment(this.dashbord[i].date).format("YYYY/MM/DD") == this.date) {
            const newItems = {
              id: this.dashbord[i].id,
              date: this.dashbord[i].date,
              location: this.dashbord[i].location,
              name: this.dashbord[i].name,
              time_start: this.dashbord[i].time_start,
              time_stop: this.dashbord[i].time_stop,
              type: this.dashbord[i].type,
            };
            this.list_select.push(newItems);
          }
          }
        }else if(this.date!=null&&this.select_room!=null){
          console.warn("Test : 3 !!")
          console.warn("Date : "+this.date)
          console.warn("Room : "+this.select_room)
          console.warn(moment(this.dashbord[0].date).format("YYYY/MM/DD")+"XXXXXXXXX"+this.date)
          for (var i = 0; i < this.dashbord.length; i++) {
            for(var j = 0; j < this.list2.length; j++){
              if(this.dashbord[i].time_start<=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].time_stop=="In Use"&&this.dashbord[i].tag_address==this.list2[j].device_address&&this.list2[j].room==this.select_room&&moment(this.dashbord[i].date).format("YYYY/MM/DD")==this.date){
                console.warn("Room 1"+this.dashbord[i].location)
                const newItems = {
                   id: this.dashbord[i].id,
                   date: this.dashbord[i].date,
                   location: this.dashbord[i].location,
                   name: this.dashbord[i].name,
                   time_start: this.dashbord[i].time_start,
                   time_stop: this.dashbord[i].time_stop,
                   type: this.dashbord[i].type,
                   };
                  this.list_select.push(newItems);
                break
              }else if(this.dashbord[i].time_start<=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].time_stop>=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].tag_address==this.list2[j].device_address&&this.list2[j].room==this.select_room&&moment(this.dashbord[i].date).format("YYYY/MM/DD")==this.date){
                console.warn("Room 2 "+this.dashbord[i].location)
                const newItems = {
                   id: this.dashbord[i].id,
                   date: this.dashbord[i].date,
                   location: this.dashbord[i].location,
                   name: this.dashbord[i].name,
                   time_start: this.dashbord[i].time_start,
                   time_stop: this.dashbord[i].time_stop,
                   type: this.dashbord[i].type,
                   };
                  this.list_select.push(newItems);
                break
              }
            }
          }
        }else if(this.date==null&&this.select_room!=null){
          console.warn("Test : 4 =!")
          for (var i = 0; i < this.dashbord.length; i++) {
            for(var j = 0; j < this.list2.length; j++){
              // console.warn("------------------------------------------------------")
              // console.warn(this.dashbord[i].time_start+" ==== "+moment(this.list2[j].scan_timestamp).format("hh:mm"))
              // console.warn(this.dashbord[i].time_stop+" ------- "+moment(this.list2[j].scan_timestamp).format("hh:mm"))
              // console.warn(this.dashbord[i].tag_address+" ==== "+this.list2[j].device_address)
              if(this.dashbord[i].time_start<=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].time_stop=="In Use"&&this.dashbord[i].tag_address==this.list2[j].device_address&&this.list2[j].room==this.select_room){
                console.warn("Room 1"+this.dashbord[i].location)
                const newItems = {
                   id: this.dashbord[i].id,
                   date: this.dashbord[i].date,
                   location: this.dashbord[i].location,
                   name: this.dashbord[i].name,
                   time_start: this.dashbord[i].time_start,
                   time_stop: this.dashbord[i].time_stop,
                   type: this.dashbord[i].type,
                   };
                  this.list_select.push(newItems);
                break
              }else if(this.dashbord[i].time_start<=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].time_stop>=moment(this.list2[j].scan_timestamp).format("hh:mm")&&this.dashbord[i].tag_address==this.list2[j].device_address&&this.list2[j].room==this.select_room){
                console.warn("Room 2 "+this.dashbord[i].location)
                const newItems = {
                   id: this.dashbord[i].id,
                   date: this.dashbord[i].date,
                   location: this.dashbord[i].location,
                   name: this.dashbord[i].name,
                   time_start: this.dashbord[i].time_start,
                   time_stop: this.dashbord[i].time_stop,
                   type: this.dashbord[i].type,
                   };
                  this.list_select.push(newItems);
                break
              }
            }
          }
        }
       
        
        console.warn(this.list_select);
      },
      async showdata() {
        // console.warn(this.date);
        this.select_room= null,
        this.list_select = this.dashbord;
      },
  },
};
</script>

<style lang="sass">
.my-sticky-header-table
  /* height or max-height is important */
  height: 490px
  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th
    /* bg color is important for th; just specify one */
    background-color: #c1f4cd
  thead tr th
    position: sticky
    z-index: 1
  thead tr:first-child th
    top: 0
  /* this is when the loading indicator appears */
  &.q-table--loading thead tr:last-child th
    /* height of all previous header rows */
    top: 48px
</style>