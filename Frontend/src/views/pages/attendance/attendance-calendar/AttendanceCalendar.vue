<template>
    <div class="demo-app">
        <div class="demo-app-main">
            <FullCalendar class="demo-app-calendar" :options="calendarOptions">
                <template v-slot:eventContent="arg">
                    <b>{{ arg.timeText }}</b>
                    <i>{{ arg.event.title }}</i>
                </template>
            </FullCalendar>
        </div>

        <Dialog v-model:visible="isModalOpen" :style="{ width: '450px' }" header="이벤트 생성" :modal="true">
            <div class="flex flex-col gap-6">
                <div>
                    <label for="title" class="block font-bold mb-3">이벤트 이름</label>
                    <InputText id="title" v-model.trim="eventData.title" required="true" autofocus :invalid="submitted && !eventData.title" fluid />
                    <small v-if="submitted && !eventData.title" class="text-red-500">이벤트 이름이 필요합니다.</small>
                </div>
                <div>
                    <label for="start" class="block font-bold mb-3">시작 날짜 및 시간</label>
                    <InputText type="datetime-local" id="start" v-model="eventData.start" required="true" fluid />
                </div>
                <div>
                    <label for="end" class="block font-bold mb-3">종료 날짜 및 시간</label>
                    <InputText type="datetime-local" id="end" v-model="eventData.end" fluid />
                </div>
                <div>
                    <label for="category" class="block font-bold mb-3">카테고리</label>
                    <Select id="category" v-model="eventData.category" :options="categories" optionLabel="label" placeholder="카테고리 선택" fluid />
                </div>
            </div>

            <template #footer>
                <Button label="닫기" icon="pi pi-times" text @click="closeModal" />
                <Button label="저장" icon="pi pi-check" @click="saveEvent" />
            </template>
        </Dialog>

        <Dialog v-model:visible="isDetailModalOpen" :style="{ width: '450px' }" header="이벤트 상세 정보" :modal="true">
            <div class="flex flex-col gap-6">
                <div v-if="selectedEvent">
                    <p><strong>이름:</strong> {{ selectedEvent.title }}</p>
                    <p><strong>카테고리:</strong> {{ selectedEvent.extendedProps.category }}</p>
                    <p><strong>시작 날짜 및 시간:</strong> {{ formatDate(selectedEvent.start) }}</p>
                    <p><strong>종료 날짜 및 시간:</strong> {{ formatDate(selectedEvent.end) }}</p>
                </div>
            </div>
            <template #footer>
                <Button label="삭제" icon="pi pi-trash" severity="danger" @click="deleteEvent(selectedEvent)" />
                <Button label="닫기" icon="pi pi-times" text @click="closeDetailModal" />
            </template>
        </Dialog>
    </div>
</template>

<script>
import dayGridPlugin from '@fullcalendar/daygrid';
import interactionPlugin from '@fullcalendar/interaction';
import timeGridPlugin from '@fullcalendar/timegrid';
import FullCalendar from '@fullcalendar/vue3';
import Panel from 'primevue/panel';
import { defineComponent, ref } from 'vue';
import { INITIAL_EVENTS } from '/src/views/pages/attendance/attendance-calendar/utils/data.js';
import { createEventId } from '/src/views/pages/attendance/attendance-calendar/utils/event-utils.js';

export default defineComponent({
    components: {
        FullCalendar,
        Panel
    },
    data() {
        return {
            calendarOptions: {
                plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
                headerToolbar: {
                    left: 'prev,next today',
                    center: 'title',
                    right: 'dayGridMonth,timeGridWeek,timeGridDay'
                },
                initialView: 'dayGridMonth',
                initialEvents: INITIAL_EVENTS,
                editable: true,
                selectable: true,
                selectMirror: true,
                dayMaxEvents: true,
                weekends: true,
                select: this.handleDateSelect,
                eventClick: this.handleEventClick,
                eventsSet: this.handleEvents
            },
            currentEvents: [],
            isModalOpen: false,
            isDetailModalOpen: false,
            selectedInfo: null,
            selectedEvent: null,
            submitted: ref(false),
            eventData: ref({
                title: '',
                start: '',
                end: '',
                category: ''
            }),
            categories: ref(['휴가', '출근', '병가', '조퇴'])
        };
    },
    methods: {
        handleDateSelect(selectInfo) {
            this.selectedInfo = selectInfo;
            this.eventData = {
                title: '',
                start: selectInfo.startStr,
                end: selectInfo.endStr,
                category: ''
            };
            this.isModalOpen = true;
        },
        handleEventClick(clickInfo) {
            this.selectedEvent = clickInfo.event;
            this.isDetailModalOpen = true;
        },
        handleEvents(events) {
            this.currentEvents = events;
        },
        saveEvent() {
            this.submitted = true;
            if (this.eventData.title.trim()) {
                let calendarApi = this.selectedInfo.view.calendar;
                calendarApi.unselect();

                const categoryColors = {
                    휴가: '#ffcccc',
                    출근: '#ccffcc',
                    병가: '#ffe6cc',
                    조퇴: '#cce6ff',
                    회의: '#cce6ff',
                    마감: '#ffe6cc',
                    리뷰: '#ffe6cc'
                };

                calendarApi.addEvent({
                    id: createEventId(),
                    title: this.eventData.title,
                    start: this.eventData.start,
                    end: this.eventData.end,
                    backgroundColor: categoryColors[this.eventData.category],
                    extendedProps: {
                        category: this.eventData.category
                    },
                    allDay: this.selectedInfo.allDay
                });

                this.isModalOpen = false;
                this.eventData = {
                    title: '',
                    start: '',
                    end: '',
                    category: ''
                };
            }
        },
        deleteEvent(event) {
            if (event) {
                event.remove();
            }
            this.isDetailModalOpen = false;
            this.selectedEvent = null;
        },
        closeModal() {
            this.isModalOpen = false;
        },
        closeDetailModal() {
            this.isDetailModalOpen = false;
        },
        formatDate(date) {
            if (!date) return 'N/A';
            return new Date(date).toLocaleString('ko-KR', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric',
                hour: '2-digit',
                minute: '2-digit'
            });
        }
    }
});
</script>
