<template>
<div class="flex justify-center items-center mt-5 p-3 bg-gray-100">
    <div class="bg-white p-8 rounded shadow">
        <h1 class="text-3xl font-bold mb-6 text-center">Ticket</h1>
        <div class="mt-2">
            <div v-if="purchase" class="bg-slate-600 p-5 rounded w-96">
                <h3 class="text-green-600 font-bold text-center">Ticket confirmed!</h3>
                <hr class="my-4">
                <h4 class="text-white">Booking Details</h4>
                <hr class="my-2">
                <div>
                    <div><strong>Date:</strong> {{ user.date }}</div>
                    <div><strong>Time:</strong> {{ user.time }}</div>
                    <div><strong>Name:</strong> {{ user.name }}</div>
                    <div><strong>Phone:</strong> {{ user.mobile }}</div>
                </div>
                <div class="mt-4">
                    <strong>Seats:</strong>
                    <span class="text-white bg-gray-800 p-1 m-1">
                        {{ user.seats.map(seat => seat.name).join(', ')}}
                    </span>
                </div>
                <div class="mt-4"><strong>Total Cost:</strong> Tk. {{ user.totalCost }}</div>
                <button @click="resetData" class="bg-pink-600 px-2 py-1 text-white rounded mt-4">Book Again</button>
            </div>
            <div v-else>
                <div class="flex justify-between">
                    <div class="mb-4">
                        <label for="date" class="block mb-2">Select Date:</label>
                        <input type="date" id="date" v-model="selectedDate" class="p-2 border rounded w-full" :min="minDate" :max="maxDate" />
                    </div>
                    <div class="mb-4">
                        <label for="time" class="block mb-2">Select Time:</label>
                        <select id="time" v-model="selectedTime" class="p-2 border rounded w-full">
                            <option value="">Select Time</option>
                            <option v-for="time in times" :key="time.start" :value="time.start">{{ time.start }}</option>
                        </select>
                    </div>
                </div>
                <div class="mb-4 flex justify-between">
                    <div v-for="(state, type) in seatStates" :key="type" class="mb-2 flex items-center">
                        <div :style="{ backgroundColor: state.color }" class="h-6 w-6 mr-2 rounded border"></div>
                        <div>{{ state.text }}</div>
                    </div>
                </div>
                <div class="flex flex-wrap justify-between">
                    <div class="w-[280px]">
                        <div class="border p-4">
                            <div class="flex justify-between mb-2">
                                <div class="border p-2 bg-sky-300 rounded">Door</div>
                                <div class="border p-2 bg-orange-300 rounded">Driver</div>
                            </div>
                            <div class="grid grid-cols-4 gap-2">
                                <div
                                    v-for="(seat, i) in filteredSeats"
                                    :key="seat.name"
                                    :class="{ 'bg-red-600 text-white': seat.type === 'sold', 'bg-gray-800': seat.type === 'booked', 'bg-slate-300': seat.type === 'available', 'bg-green-500': seat.type === 'selected', 'cursor-pointer': seat.type !== 'sold' && seat.type !== 'booked' }"
                                    class="seat border p-2 text-center rounded"
                                    @click="seatClick(i)"
                                >
                                    {{ seat.name }}
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="w-[400px]">
                        <div v-if="selectedSeats.length === 0" class="p-2 flex items-center justify-center h-full">
                            <strong>No seats selected<br>select some seats first</strong>
                        </div>
                        <div v-else class="flex-1 pl-6">
                            <div class="mb-4">
                                <strong>Selected Seats</strong>
                                <table class="w-full mt-2 border text-center">
                                    <thead>
                                        <tr>
                                            <th class="border">Seat</th>
                                            <th class="border">Price</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr v-for="seat in selectedSeats" :key="seat.name">
                                            <td class="border">{{ seat.name }}</td>
                                            <td class="border">Tk. {{ seat.price }}</td>
                                        </tr>
                                        <tr v-if="appliedCoupon">
                                            <th class="border">Discount</th>
                                            <th class="border">Tk. -{{ appliedCoupon.discount }}</th>
                                        </tr>
                                        <tr>
                                            <th class="border">Total</th>
                                            <th class="border">Tk. {{ totalCost }}</th>
                                        </tr>
                                    </tbody>
                                </table>
                                <div class="mt-2">
                                    <div v-if="!appliedCoupon">
                                        <p>Have a coupon?</p>
                                        <div class="flex justify-around">
                                            <input type="text" v-model="couponCode" placeholder="Enter valid Coupon Code" class="ml-2 p-1 border rounded">
                                            <button @click="applyCoupon" class="mt-2 px-4 py-2 bg-blue-500 text-white rounded">Apply</button>
                                        </div>
                                    </div>
                                    <p v-else class="text-center text-green-500">Applied Coupon: {{ appliedCoupon.code }}</p>
                                </div>
                            </div>
                        <div>
                        <input type="text" placeholder="Name" v-model="user.name" class="w-full p-2 mb-2 border rounded" />
                        <input type="text" placeholder="Mobile" v-model="user.mobile" class="w-full p-2 mb-2 border rounded" />
                        </div>
                            <button @click="purchaseNow" class="bg-indigo-500 py-2 rounded mt-2 w-full text-white">Purchase Now</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import showAlert from '../helpers/alert';

const purchase = ref(false);
const user = ref({ date: '', time: '', name: '', mobile: '', seats: [], totalCost: 0 });
const selectedDate = ref('');
const selectedTime = ref('');
const appliedCoupon = ref(null);
const couponCode = ref('');

const minDate = ref('');
const maxDate = ref('');
const today = new Date();
const nextWeek = new Date(today);
nextWeek.setDate(nextWeek.getDate() + 6);
minDate.value = today.toISOString().split('T')[0];
maxDate.value = nextWeek.toISOString().split('T')[0];

const coupons = ref([
    { code: "100TAKAOFF", discount: 100 },
    { code: "200TAKAOFF", discount: 200 }
]);

const times = ref([
    { start: "05:25 AM" },
    { start: "06:30 AM" },
    { start: "07:45 AM" },
    { start: "08:00 AM" },
]);

const seatStates = {
    sold: { text: "Sold", color: "#ff0000" },
    available: { text: "Available", color: "#dedede" },
    booked: { text: "Booked", color: "#313131" },
    selected: { text: "Selected", color: "#00ff00" }
};

const initialSeats = [
    { name: "A1", type: "available", price: 500 },
    { name: "A2", type: "available", price: 500 },
    { name: "A3", type: "available", price: 500 },
    { name: "A4", type: "available", price: 500 },
    { name: "B1", type: "available", price: 450 },
    { name: "B2", type: "available", price: 450 },
    { name: "B3", type: "available", price: 450 },
    { name: "B4", type: "available", price: 450 },
    { name: "C1", type: "available", price: 500 },
    { name: "C2", type: "available", price: 500 },
    { name: "C3", type: "available", price: 500 },
    { name: "C4", type: "available", price: 500 },
    { name: "D1", type: "available", price: 400 },
    { name: "D2", type: "available", price: 400 },
    { name: "D3", type: "available", price: 400 },
    { name: "D4", type: "available", price: 400 },
    { name: "E1", type: "available", price: 300 },
    { name: "E2", type: "available", price: 300 },
    { name: "E3", type: "available", price: 300 },
    { name: "E4", type: "available", price: 300 },
    { name: "F1", type: "available", price: 300 },
    { name: "F2", type: "available", price: 300 },
    { name: "F3", type: "available", price: 300 },
    { name: "F4", type: "available", price: 300 }
];

const seatsByDateTime = ref({});

const seats = ref([...initialSeats]);

const seatClick = (i) => {
    if(!selectedDate.value) {
        showAlert('error', "Please select date first");
        return;
    }
    if(!selectedTime.value) {
        showAlert('error', "Please select time first");
        return;
    }
    const clickedSeat = seats.value[i];

    if (clickedSeat.type === 'sold') {
        showAlert('error', "This seat is already sold");
        return;
    }

    if (clickedSeat.type === 'booked') {
        showAlert('error', "This seat is already booked");
        return;
    }

    if (clickedSeat.type === 'available' && selectedSeats.value.length >= 4) {
        showAlert('error', "You can't select more than 4 seats");
        return;
    }

    clickedSeat.type = clickedSeat.type === 'selected' ? 'available' : 'selected';

    filteredSeats.value[i] = clickedSeat;
};

const selectedSeats = computed(() => seats.value.filter(seat => seat.type === 'selected'));

const totalCost = computed(() => {
    let total = selectedSeats.value.reduce((sum, seat) => sum + seat.price, 0);
    if (appliedCoupon.value) total -= appliedCoupon.value.discount;
    return total;
});

const purchaseNow = () => {
    if (!selectedSeats.value.length) {
        showAlert('error', "No seats selected");
        return;
    }
    if (!user.value.name) {
        showAlert('error', "Name is required");
        return;
    }
    if (!user.value.mobile) {
        showAlert('error', "Mobile is required");
        return;
    }

    const key = `${selectedDate.value}-${selectedTime.value}`;
    if (!seatsByDateTime.value[key]) {
        seatsByDateTime.value[key] = JSON.parse(JSON.stringify(initialSeats));
    }

    user.value.date = selectedDate.value;
    user.value.time = selectedTime.value;
    user.value.seats = selectedSeats.value;
    user.value.totalCost = totalCost.value;

    selectedSeats.value.forEach(seat => {
        const seatIndex = seatsByDateTime.value[key].findIndex(s => s.name === seat.name);
        if (seatIndex !== -1) seatsByDateTime.value[key][seatIndex].type = 'sold';
    });

    seats.value = JSON.parse(JSON.stringify(seatsByDateTime.value[key]));
    purchase.value = true;
};

const resetData = () => {
    purchase.value = false;
    user.value = { date: '', time: '', name: '', mobile: '', seats: [], totalCost: 0 };
    selectedDate.value = '';
    selectedTime.value = '';
    appliedCoupon.value = null;
    couponCode.value = '';
    const key = `${selectedDate.value}-${selectedTime.value}`;
    if (seatsByDateTime.value[key]) {
        seatsByDateTime.value[key].forEach(seat => {
            if (seat.type === 'selected') seat.type = 'sold';
        });
    }
    seats.value = JSON.parse(JSON.stringify(initialSeats));
};

const applyCoupon = () => {
    const coupon = coupons.value.find(c => c.code === couponCode.value);
    if (coupon) {
        appliedCoupon.value = coupon;
        showAlert('success', "Coupon applied successfully");
    } else {
        showAlert('error', "Invalid coupon code");
    }
};

const filteredSeats = computed(() => {
    if (!selectedDate.value || !selectedTime.value) {
        return seats.value;
    }

    const key = `${selectedDate.value}-${selectedTime.value}`;
    if (!seatsByDateTime.value[key]) {
        seatsByDateTime.value[key] = JSON.parse(JSON.stringify(initialSeats));
    }
    return seatsByDateTime.value[key];
});

watch([selectedDate, selectedTime], () => {
    const key = `${selectedDate.value}-${selectedTime.value}`;
    if (seatsByDateTime.value[key]) {
        seats.value = JSON.parse(JSON.stringify(seatsByDateTime.value[key]));
    } else {
        seats.value = JSON.parse(JSON.stringify(initialSeats));
    }
});
</script>

<style scoped>
.seat {
    margin: 4px;
    min-height: 40px;
    font-size: 15px;
    width: 40px;
}

.seat:hover {
    font-weight: bold;
}

.seat:nth-child(4n -2) {
    margin-left: -5px;
}

.seat:nth-child(4n -1) {
    margin-left: 15px;
}
</style>