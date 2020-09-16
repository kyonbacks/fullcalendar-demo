<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div class="swapper">
    <div class="controller-row">
      <el-button @click="getEvent">获取数据(请打开f12查看)</el-button>
    </div>
    <el-row :gutter="20">
      <el-col :span="6">
        <h3>事件列表</h3>
        <div class="msg-list" ref="list">
          <div class="msg-list-item" v-for="(item,index) in dropEvents" :key="index" :data-index="index">
            <div class="msg-item" :style="{backgroundColor:item.color}">{{item.title}}</div>
          </div>
        </div>
        <div>
          <el-checkbox v-model="checked">拖拽后移除</el-checkbox>
        </div>
        <br>
        <h3>可以创建事件</h3>
        <el-form :model="addform">
          <el-form-item label="活动名称">
            <el-input v-model="addform.title" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="选择颜色">
            <color-select :colorList="colorList" v-model="addform.color"></color-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="defauleEventClick">确认新增</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="18">
        <FullCalendar ref="fullCalendar" :options="calendarOptions">
          <template v-slot:eventContent='arg'>
            <b>{{ arg.timeText }}</b>
            <i>{{ arg.event.title }}</i>
          </template>
        </FullCalendar>
      </el-col>
    </el-row>
    <el-dialog title="请输入内容" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="活动名称">
          <el-input v-model="form.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="选择颜色">
          <color-select :colorList="colorList" v-model="form.color"></color-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addElement">确 定</el-button>
      </div>
    </el-dialog>
  </div>

</template>

<script>
import 'bootstrap/dist/css/bootstrap.css';
import '@fortawesome/fontawesome-free/css/all.css';
import FullCalendar from '@fullcalendar/vue';
import dayGridPlugin from '@fullcalendar/daygrid';
import timeGridPlugin from '@fullcalendar/timegrid';
import bootstrapPlugin from '@fullcalendar/bootstrap';
import interactionPlugin,{ Draggable } from '@fullcalendar/interaction';
import { INITIAL_EVENTS, createEventId } from '@/utils/event-utils'
import ColorSelect from "@/components/colorSelect";
export default {
  name: "Home",
  components: {ColorSelect, FullCalendar},
  data(){
    return {
      colorList:['#007bff','#009378','#e16123','#2bb3c0','#ff4040','#353535','#ccc'],
      dialogFormVisible:false,
      currentEvents: [],//已经存在的事件
      dropEvents:[
        {id: createEventId(),title:'测试事件',color:'#007bff'}
      ],//可以拖动的事件
      checked:false,//是否拖拽后移除
      form:{ //弹出层操作form
        title:"",
        color:"#007bff",
        info:undefined
      },
      addform:{ //左边栏form
        title:"",
        color:"#007bff"
      },
      calendarOptions:{
        plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin, bootstrapPlugin],
        locale: 'zh',
        themeSystem: 'bootstrap',
        buttonText: {
          today: '今天',
          month: '月',
          week: '周',
          day: '日'
        },
        headerToolbar: {
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,timeGridWeek,timeGridDay'
        },
        initialView: 'dayGridMonth',//加载默认视图
        initialEvents: INITIAL_EVENTS,//初始值
        droppable: true,//是否可拖动
        editable: true,//确定是否可以修改日历上的事件。
        selectable: true,//允许用户通过单击和拖动来突出显示多天或多个时隙。
        selectMirror: true,//用户拖动时是否绘制“占位符”事件。
        dayMaxEvents:true,//给定日期内的最大活动数，不包括+ more链接。其余的将显示在弹出窗口中。如果true已指定，则事件数将限制为日单元格的高度。
        weekends:true,//是否显示周末，设为false则不显示周六和周日。默认值为true
        select: this.handleDateSelect,//点击事件
        disableResizing:true,
        eventClick: this.handleEventClick,//事件的点击事件
        eventsSet: this.handleEvents,//事件变化的监听
        events: [],
        drop:this.handleEventDrop,
        /* 这些发生时，您可以更新远程数据库:
        eventAdd:
        eventChange:
        eventRemove:
        */
      }
    }
  },
  methods: {
    handleDateClick: function(arg) {
      this.dialogFormVisible = true;
      this.form.date = arg.dateStr;
    },
    //点击日期
    handleDateSelect(selectInfo){
      this.dialogFormVisible = true;
      this.form.selectInfo = selectInfo;
    },
    //添加弹窗
    addElement(){
      this.dialogFormVisible = false;
      let title = this.form.title;
      let selectInfo = this.form.selectInfo;
      let calendarApi = selectInfo.view.calendar
      calendarApi.unselect() // clear date selection
      if (title) {
        calendarApi.addEvent({
          id: createEventId(),
          title,
          start: selectInfo.startStr,
          end: selectInfo.endStr,
          allDay: selectInfo.allDay,
          color:this.form.color
        })
      }
    },
    //事件的点击事件
    handleEventClick(clickInfo) {
      if (confirm(`请确认删除事件： '${clickInfo.event.title}'`)) {
        clickInfo.event.remove()
      }
    },
    //每次events变化时的change
    handleEvents(events) {
      this.currentEvents = events
    },
    //左边栏新增拖动事件的确定事件
    defauleEventClick(){
      this.dropEvents.push({id: createEventId(),title:this.addform.title,color:this.addform.color})
    },
    //左边栏拖动到日历的事件
    handleEventDrop(info){
      if(this.checked){
        const index = info.draggedEl.dataset.index;
        this.dropEvents.splice(index,1);
      }
    },
    getEvent(){
      let x = this.$refs.fullCalendar.getApi();
      console.log(x.getEvents());
      console.log(this.currentEvents);
    },

  },
  mounted(){
    let that = this;
    //初始化拖动
    new Draggable(this.$refs.list, {
      itemSelector: '.msg-list-item',
      eventData(eventEl) {
        const index = eventEl.dataset.index;
        const data = that.dropEvents[index];
        return data
      }
    });
  }
}
</script>

<style scoped lang="less">
  div{
    text-align: left;
  }
  .controller-row{
    margin-bottom: 10px;
  }
  .swapper{
    padding: 10px;
  }
  .msg-list{
    .msg-list-item{
      margin-bottom: 10px;
    }
    .msg-item{
      height: 40px;
      line-height: 40px;
      background-color: deepskyblue;
      color: white;
      text-align: center;
      cursor: move;
    }
  }
</style>
