<template>
<div>
    <div>
        <p>Please enter your details</p>
        <label>Name</label><br>
        <input v-model="firstName"><br>
        <label>Phone Number</label><br>
        <input v-model="phoneNo"><br>
        <button @click='placeOrder()'>Place Order</button>
    </div>

    <ul>
        <li v-bind:key="lesson.key" v-for="lesson in getLessons()"> 
            <img v-bind:src="lesson.image"> 
            <h1>{{lesson.subject}}</h1>
            <p>location: {{lesson.location}}</p>
            <p>Price: £{{lesson.price}}</p>
            <button @click='removeLesson(lesson._id)'><span v-bind:class="lesson.icon"></span> Remove Lesson</button>
        </li>
    </ul>

</div>
</template>

<script>
export default {
    name: "CheckoutPage",
    props: ['cart','lesson'],
    data() {
        return {
            firstName: '',
            phoneNo: ''
        };
    },
    methods: {
        removeLesson(lessonid){
            this.$emit('removeItem',lessonid)
        },
        getLessons(){
            
            let lessonDetails = []
            for (let id of this.cart){
              for (let _lesson of this.lesson){
                if(id === _lesson._id){
                  lessonDetails.push(_lesson)
                }
              }
            }
            return lessonDetails;
        },
        placeOrder(){
            let Spaces = [];
            let IDs = [];
            let lessonData = JSON.parse(JSON.stringify(this.lesson))
            this.cart.forEach(function(cartItem){
                lessonData.forEach(function(lessonItem){
                if(cartItem === lessonItem._id && IDs.includes(cartItem) == false ){
                    let temp = lessonItem.subject + "-" + lessonItem.location
                    Spaces.push({Lesson: temp,Ordered: 0})
                    IDs.push(cartItem);
                }
                });
            });
            this.cart.forEach(function(cartItem){
                lessonData.forEach(function(lessonItem){
                if(cartItem === lessonItem._id){
                    let temp = lessonItem.subject + "-" + lessonItem.location
                    for(let i = 0; i < Spaces.length; i++){
                        if(temp == Spaces[i].Lesson){
                            Spaces[i].Ordered +=1;
                        }
                    }
                }
                });
            });
            console.log(Spaces);
            console.log(IDs);
            const order = {name: this.firstName, phone_number: this.phoneNo, subject: IDs, space: Spaces};
            fetch('https://cw2individual.herokuapp.com/collection/orders/', { 
                method: 'POST',
                headers: {
                'Content-Type': 'application/json',
                },
                body: JSON.stringify(order),
            })
            .then(response => response.json())
            .then(responseJSON => {
                console.log('Success:', responseJSON);
            });
            for (let i = 0; i < this.cart.length; i++) {
                for(let y = 0; y < this.lesson.length; y++){
                if(this.cart[i] === this.lesson[y]._id){
                    let updateSpaces = { "spaces": (this.lesson[y].space) };
                    fetch("https://cw2individual.herokuapp.com/collection/lessons/" + this.cart[i], {
                    method: "PUT", 
                    headers: {
                        "Content-Type": "application/json", 
                    },
                    body: JSON.stringify(updateSpaces),
                    });
                }
                }
            }
            
            alert("Thank you for your purchase :)");
            this.$emit('clearCart')
            fetch('https://cw2individual.herokuapp.com/collection/lessons/')
            .then(response => response.json())
            .then(data => (this.searchLessons = data))
        },
    },
}
</script>