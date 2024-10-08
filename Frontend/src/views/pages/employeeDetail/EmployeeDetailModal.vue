<template>
    <Dialog :visible.sync="localVisible" header="사원 상세 정보" :modal="true" :closable="true" @update:visible="handleVisibilityUpdate" :style="{ width: '500px', maxWidth: '90vw' }">
        <div class="employee-details-container">
            <div class="employee-details">
                <p><strong>이름 :</strong> {{ employee.employeeName }}</p>
                <p><strong>부서 :</strong> {{ employee.deptName }}</p>
                <p><strong>팀 :</strong> {{ employee.teamName }}</p>
                <p><strong>직책 :</strong> {{ employee.positionName }}</p>
                <p><strong>사번 :</strong> {{ employee.employeeId }}</p>
                <p><strong>입사일 :</strong> {{ formatDate(new Date(employee.joinDate)) }}</p>
            </div>
            <div class="photo-upload-container">
                <img :src="photoUrl" alt="증명사진" class="employee-photo" />
            </div>
        </div>
    </Dialog>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
    employee: Object,
    visible: Boolean
});

const emit = defineEmits(['update:visible']);

const localVisible = ref(props.visible);
const employee = ref(props.employee);
const photoUrl = ref('');

watch(
    () => props.visible,
    (newVal) => {
        localVisible.value = newVal;
    }
);

function handleVisibilityUpdate(newVal) {
    localVisible.value = newVal;
    emit('update:visible', newVal);
}

watch(
    () => props.employee,
    (newVal) => {
        employee.value = newVal;
        if (newVal && newVal.profileImageUrl) {
            photoUrl.value = newVal.profileImageUrl;
        } else {
            photoUrl.value = 'https://via.placeholder.com/150';
        }
    },
    { immediate: true }
);

function formatDate(date) {
    if (!(date instanceof Date) || isNaN(date.getTime())) {
        return '';
    }
    let formattedDate = date.toLocaleDateString(); // 기본 날짜 형식
    if (formattedDate.endsWith('.')) {
        formattedDate = formattedDate.slice(0, -1); // 마지막에 . 이 있으면 제거
    }
    return formattedDate;
}
</script>

<style scoped>
.employee-details-container {
    display: flex;
    align-items: center;
    padding: 1.5rem;
    background-color: #ffffff;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    border-radius: 8px;
}

.employee-details {
    flex: 1;
    padding-left: 20px; /* 왼쪽 여백 추가 */
}

.photo-upload-container {
    margin-left: 20px;
}

.employee-photo {
    width: 150px;
    height: 150px;
    object-fit: cover;
    border-radius: 4px;
}

.employee-details p {
    margin: 0.5rem 0;
    font-size: 1rem;
    line-height: 1.6;
    color: #333;
}

strong {
    color: #2c3e50;
}
</style>
