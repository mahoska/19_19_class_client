<template>
<div class="cart">
  <div class="bg-danger" id="err">{{isDelCartRec}}</div>
  <div class="cart_info">{{isUpdCartRec}}</div>
  <table class="table table-bordered cart-table">
    <tr>
      <th>Active</th>
      <th>Book name</th>
      <th>Price</th>
      <th>Count</th>
      <th>Sum</th>
    </tr>
    <tr v-for="book in cart" :key="book.id">
      <td>
        <div class="checkbox">
          <label>
            <input type="checkbox" :value="book.id"  :id="book.id" checked  @change="checkBook(book.id,book.count)">
          </label>
        </div>
      </td>
      <td>
          <router-link  :to="'/book/'+book.id" class="btn btn-auth" >
            {{ book.name }}
          </router-link>
      </td>
      <td>{{book.new_price_str}}</td>
      <td>
        <div>
						<div class="btn min_btn inline" id="minus" @click="changeCount(false,book.id)">-</div>
						<div class="count inline">{{book.count}}</div>
						<div class="btn min_btn  inline" id="plus" @click="changeCount(true,book.id)">+</div>
				</div>
      </td>
      <td>{{book.sum_str}}</td>
    </tr>
  </table>
  <div class="sum">
    <p><span>Total:</span> {{total_sum_str}} GRN </p>
    <p><span>Discount:</span> {{sum_client_discount_str}} GRN ({{client_discount}}%)</p>
    <p><span>Sum: </span>{{total_sum_with_discount_str}} GRN </p>
  </div>
  <button class="btn btn-def-light" @click="update">Update</button>
   <router-link  :to="'/payment'" class="btn btn-def"  >
     Checkout
   </router-link>
</div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'cart',
  data () {
    return {
      cart: [],
      isDelCartRec: "",
      isUpdCartRec: "",
      total_sum: 0,
      total_sum_with_discount: 0,
      client_discount: 0,
      sum_client_discount: 0,
      total_sum_str: "",
      sum_client_discount_str: "",
      total_sum_with_discount_str: "",
      client: {}
    }
  },
  created(){
    var self = this
    

      if(localStorage['hash_log'] && localStorage['user']){
        axios.get('http://localhost:88/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
        //axios.get('http://192.168.0.15/~user15/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
              .then(function (response) {
                var client_id = parseInt(response.data.data)
                if(client_id>0){
                  axios.get('http://localhost:88/BOOK_SHOP/client/api/auth/'+ client_id, self.config)
                  //axios.get('http://192.168.0.15/~user15/BOOK_SHOP/client/api/auth/'+ client_id , self.config)
                  .then(function (response) {
                     self.client = response.data.data
                    self.client_discount =  parseFloat( self.client.discount)
                      axios.get('http://localhost:88/BOOK_SHOP/client/api/cart/'+ client_id +'/1', self.config)
                      //axios.get('http://192.168.0.15/~user15/BOOK_SHOP/client/api/cart/'+ client_id +'/1', self.config)
                      .then(function (response) {
                        //console.log(response.data.data)
                        self.cart = response.data.data
                        self.cart.forEach(function(element){
                          element.book_discount = parseFloat(element.book_discount)
                          //element.client_discount = parseFloat(element.client_discount)
                          element.price = parseFloat(element.price)
                          //self.client_discount = element.client_discount

                          if(element.book_discount > 0 && element.book_discount < 100){
                            element.new_price = element.price - element.price*element.book_discount/100
                            element.new_price = Math.round(element.new_price * 100) / 100 
                          }else{
                            element.new_price = element.price
                          }

                          element.new_price_str  = element.new_price.toFixed(2)
                          element.sum = element.count*element.new_price
                          element.sum = Math.round(element.sum * 100) / 100 
                          element.sum_str  = element.sum.toFixed(2)
                          self.total_sum  += element.sum
                          self.total_sum_str  = self.total_sum.toFixed(2)
                        
                        },self)

                        if(self.client_discount > 0 && self.client_discount < 100){
                          self.sum_client_discount = self.total_sum*self.client_discount/100
                          self.sum_client_discount = Math.round(self.sum_client_discount * 100) / 100 
                          self.sum_client_discount_str = self.sum_client_discount.toFixed(2)
                          self.total_sum_with_discount = self.total_sum -  self.sum_client_discount
                        }else{
                          self.total_sum_with_discount = self.total_sum  
                        }
                        self.total_sum_with_discount_str = self.total_sum_with_discount.toFixed(2)
                      });

                }).catch(function (error) {
                  console.log(error) 
                });

                } 
              });
      }

      
  },
  computed:{
   
  },
  methods:{
   checkBook: function(book_id,book_count){
      var self = this
      this.isDelCartRec = ""
      var el = document.getElementById(book_id)
      if(!el.checked){
        var del = -1
        for(var i=0; i<this.cart.length; i++){
          if(this.cart[i].id==book_id){
             del=i
            if(localStorage['hash_log'] && localStorage['user']){
            axios.get('http://localhost:88/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
            //axios.get('http://192.168.0.15/~user15/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
              .then(function (response) {
                var client_id = parseInt(response.data.data)
                if(client_id>0){ 
                  if (confirm("Are you sure you want to cancel this order?")) {
                      //axios.delete('http://localhost:88/BOOK_SHOP/client/api/cart/'+book_id, self.config)
                      //axios.delete('http://192.168.0.15/~user15/BOOK_SHOP/client/api/cart/'+book_id+'/'+client_id, self.config)
                      axios.delete('http://192.168.0.15/~user15/BOOK_SHOP/client/api/cart/'+book_id, self.config)
                          .then(function (response) {
                            //console.log(self.cart);
                          if(response.data.data==1){
                              self.cart.splice(del, 1);
                              self.$emit('changeCountTotal',book_count,false)
                          }
                      })
                      .catch(function (error) {
                        console.log(error);
                      });
                
                  } 
                }else{
                  self.isDelCartRec = "for this action you need to register"
                    setTimeout(function () {
                      self.isDelCartRec = ""
                    },1500);
                  self.$emit('setlogout')     
                }
                })
                .catch(function (error) {
                  self.$emit('setlogout') 
                });
            }
          }
        }
      }
    },
    changeCount(flag,book_id){
      for(var i=0; i<this.cart.length; i++){
        if(this.cart[i].id==book_id){
          if(flag){
            this.cart[i].count++
            this.cart[i].sum += this.cart[i].new_price
            this.total_sum += this.cart[i].new_price
            this.cart[i].upfl = true
            this.$emit('changeCountTotal', 1, true )
          }
          else{
            if(this.cart[i].count >= 2){
              this.cart[i].count--
              this.cart[i].sum -= this.cart[i].new_price
              this.total_sum -= this.cart[i].new_price
              this.cart[i].upfl = true
              this.$emit('changeCountTotal', 1, false )
            }
          }
          this.cart[i].sum_str = this.cart[i].sum.toFixed(2)
          this.total_sum_str = this.total_sum.toFixed(2)

         if(this.client_discount > 0 && this.client_discount < 100){
              this.sum_client_discount = this.total_sum*this.client_discount/100
              this.sum_client_discount = Math.round(this.sum_client_discount * 100) / 100
              this.sum_client_discount_str = this.sum_client_discount.toFixed(2)
              this.total_sum_with_discount = this.total_sum -  this.sum_client_discount
              this.total_sum_with_discount_str = this.total_sum_with_discount.toFixed(2)
         }
        }
      }
    },

    update(){
      var self = this
      var up = 0, rup = 0
      if(localStorage['hash_log'] && localStorage['user']){
            axios.get('http://localhost:88/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
            //axios.get('http://192.168.0.15/~user15/BOOK_SHOP/client/api/auth/'+localStorage['hash_log']+"/"+localStorage['user'], this.config)
              .then(function (response) {
                var client_id = parseInt(response.data.data)
                if(client_id>0){ 
                    self.cart.forEach(function(el) {
                      if(el.upfl && el.upfl == true){
                        up++
                      //axios.put('http://192.168.0.15/~user15/BOOK_SHOP/client/api/cart/', 
                      axios.put('http://localhost:88/BOOK_SHOP/client/api/cart/', 
                        {
                          book_id: el.id,
                          count: el.count,
                          client_id: client_id
                        }, self.config)
                        .then(function (response) {
                          if(response.data.data == '1'){
                            rup++
                            el.upfl = false
                           
                            if(up>0 && rup>0){
                              if(rup == up){
                                self.isUpdCartRec = "data sourses successfully updated"
                              }else{
                                self.isUpdCartRec = "data sourses successfully updated partially. Check the data by updating the page."
                              }   
                                setTimeout(function () {
                                  self.isUpdCartRec = ""
                                },1500);
                              }
                          }
                        })
                        .catch(function (error) {
                            console.log(error);
                        });
                      }
                    }, self);
                    
                }else{
                  self.isDelCartRec = "for this action you need to register"
                    setTimeout(function () {
                      self.isDelCartRec = ""
                    },1500);
                  self.$emit('setlogout')     
                }
                })
                .catch(function (error) {
                  self.isDelCartRec = "for this action you need to register"
                    setTimeout(function () {
                      self.isDelCartRec = ""
                    },1500);
                  self.$emit('setlogout')  
                });
      }else{
        this.isDelCartRec = "for this action you need to register"
          setTimeout(function () {
            this.isDelCartRec = ""
          },1500);
        this.$emit('setlogout')  
      }
    },

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.cart{
min-height: 500px;
}

.inline{
	display:inline-block;
}

.min_btn{
  background-color:#B0C9E2;
	color: black;
}

    table {
			font-family: "Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
      margin: 50px auto;
			font-size: 14px;
			background: white;
      color: black;
			max-width: 70%;
			width: 70%;
			border-collapse: collapse;
			text-align: left;
			/*background: url("../assets/blurry.jpg") no-repeat;*/
			/*background-position: 100% 55px;*/
		}
    th {
		  font-weight: normal;
		  color: #339;
		  padding: 10px 12px;
		}
		td {
		 background-image: url("../assets/back.png");
		 filter:alpha(opacity=60);
		 opacity:0.6;
		  color: #669;
		  border-top: 1px solid white;
		  padding: 10px 12px;
		}
	  tr:hover td {
		  /*background-image: url("back.png");*/
		  background: transparent;/*Устанавливает прозрачный фон. */
		}

  .btn-def, .btn-def-light{
    width: 100%;
    color: white; 
    margin-top: 10px;
    width: 100px;
    background-color: #2C3E50;
    margin-right: 50px;
  }

.btn-def-light{
background-color: #4A838C;
}

.cart_info{
  color: white;
  margin-top: 50px;
  font-weight: bold;
  font-size: 17px;
}

.sum{
  color:#A2A2C1;
  width: 30%;
  float:right;
  text-align: left;
  margin-right: 150px;

}

.sum span{
  font-weight: bold;
  margin-right: 50px;
  
}

</style>
