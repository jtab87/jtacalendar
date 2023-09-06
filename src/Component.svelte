<script>
  import { getContext, onMount } from "svelte";
  import daygridPlugin from "@fullcalendar/daygrid";
  import timeGridPlugin from "@fullcalendar/timegrid";
  import listPlugin from "@fullcalendar/list";
  //import { onMount } from "svelte";
  import { Calendar } from "@fullcalendar/core";
  import adaptivePlugin from "@fullcalendar/adaptive";
  import interactionPlugin from "@fullcalendar/interaction";
  import resourceTimelinePlugin from "@fullcalendar/resource-timeline";
  import frLocale from "@fullcalendar/core/locales/fr";

  let calendarEl;

  export let licence = "CC-Attribution-NonCommercial-NoDerivatives";
  export let styleCal = "background-color:white;color:black;";
  export let slotMinTime;
  export let slotMaxTime;
  export let editable;
  export let timeLine = false;

  export let dataProvider;
  export let champId;
  export let champTitre;
  export let champDebut;
  export let champFin;
  export let champTip;

  export let dataProvider1;
  export let champId1;
  export let champTitre1;
  export let champGroupId1;
  export let champEventColor1;
  export let titreGroupes1;
  export let larCol1;

  export let clickEvent;
  export let clickDate;
  export let eventChange;

  const { styleable, Provider } = getContext("sdk");
  const component = getContext("component");

  let calendar;
  let eventsList = [];
  let resourcesList = [];

  let jtacal = {
    getInstance: function () {
      return calendar;
    },
    addEvent: function (event) {
      let evt = {
        id: event.id,
        start: event.start,
        end: event.end,
        title: event.title,
        extendedProps: {},
      };
      if (event.resourceId) {
        evt.resourceId = event.resourceId;
        evt.extendedProps.resourceId = event.resourceId;
      }
      if (event.comm) evt.extendedProps.comm = event.comm;
      if (event.allDay) evt.allDay = event.allDay;
      if (event.color) evt.color = event.color;
      calendar.addEvent(evt);
    },
    delEvent: function (id) {
      let evt = calendar.getEventById(id);
      if (evt) {
        evt.remove();
        return true;
      }
      return false;
    },
    updateEvent: function (event) {
      this.delEvent(event.id);
      this.addEvent(event);
    },
    getResourceId: function (event) {
      var evt = calendar.getEventById(event.id);
      var resourceId = "";
      if (evt) {
        var resources = evt.getResources();
        resourceId = resources[0] ? resources[0].id : "";
      }
      return resourceId;
    },
  };

  $: dataContext = {
    jtacal,
  };

  let options = {
    plugins: [
      adaptivePlugin,
      interactionPlugin,
      daygridPlugin,
      listPlugin,
      timeGridPlugin,
      resourceTimelinePlugin,
    ],
    schedulerLicenseKey: licence, //"CC-Attribution-NonCommercial-NoDerivatives",
    locales: [frLocale],
    locale: "fr",
    slotMinTime: slotMinTime,
    slotMaxTime: slotMaxTime,
    now: new Date(), //'2018-02-07',
    editable: editable, // enable draggable events
    aspectRatio: 2,
    //height: "100%",
    weekends: true,
    scrollTime: "00:00", // undo default 6am scrollTime
    headerToolbar: {
      left: "today prev,next",
      center: "title",
      right: timeLine
        ? "resourceTimelineDay,resourceTimelineWeek,resourceTimelineMonth"
        : "dayGridMonth,dayGridWeek,timeGridDay",
    },
    initialView: timeLine ? "resourceTimelineDay" : "dayGridMonth",
    events: eventsList,
    eventClick: (info) => {
      clickEvent({
        value: info.event,
        view: info.view.type,
      });
    },
    dateClick: (info) => {
      clickDate({
        value: info.date,
        view: info.view.type,
        resource: info.resource,
      });
    },
    datesSet: (info) => {
      /*xxx({
        start: info.startStr,
        end: info.endStr,
      });*/
      //console.log("changement : debut=" + info.startStr);
    },
    eventChange: function (info) {
      var resources = info.event.getResources();
      var resourceId = resources[0] ? resources[0].id : "";
      eventChange({
        event: info.event,
        oldEvent: info.oldEvent,
        resourceId: resourceId,
      });
    },
    eventDidMount: function (info) {
      if (info.event.extendedProps && info.event.extendedProps.comm) {
        info.el.setAttribute("title", info.event.extendedProps.comm);
      }
    },
  };

  if (timeLine) {
    options.resourceAreaHeaderContent = titreGroupes1; //"Salles";
    options.resourceGroupField = champGroupId1;
    (options.resourceAreaWidth = larCol1), //"20%",
      (options.resources = resourcesList);
  }

  // Test
  $: {
    if (calendar && options && champTitre) {
      console.log("refresh calendar sur modif champtitre");
    }
  }

  onMount(async () => {
    if (dataProvider.rows && champTitre && champId && champDebut && champFin) {
      eventsList =
        dataProvider?.rows?.map((x) => {
          let evt = {
            id: x[champId],
            resourceId: x["resourceId"] || "1",
            start: x[champDebut],
            end: x[champFin],
            title: x[champTitre],
            extendedProps: {
              comm: x[champTip] || x[champTitre],
              resourceId: x["resourceId"] || "1",
            },
            allDay: x["allDay"] || false,
          };
          if (x["color"]) evt.color = x["color"];
          return evt;
        }) ?? [];
      options.events = eventsList;
    }
    if (
      dataProvider1 &&
      champId1 &&
      champTitre1 &&
      champGroupId1 &&
      champEventColor1
    ) {
      resourcesList =
        dataProvider1?.rows?.map((x) => {
          return {
            id: x[champId1],
            groupId: x[champGroupId1],
            title: x[champTitre1],
            eventColor: x[champEventColor1] || "blue",
          };
        }) ?? [];
      options.resources = resourcesList;
    }
    initCalendar();
    return () => {
      calendar && calendar.destroy();
    };
  });

  function initCalendar() {
    calendar = new Calendar(calendarEl, options);
    calendar.render();
  }
  // "background-color:white;color:black;"
</script>

<div use:styleable={$component.styles}>
  <Provider data={dataContext}>
    <div bind:this={calendarEl} style={styleCal} />
    <slot />
  </Provider>
</div>
