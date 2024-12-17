
1. Backend Setup (Laravel)
Step 1: Install Laravel
Create a new Laravel project:
 bash
composer create-project laravel/laravel hotel-food-app
cd hotel-food-app
Step 2: Database Configuration
Update `.env` file with your database credentials:
dotenv
DB_DATABASE=hotel_food_db
DB_USERNAME=your_db_username
DB_PASSWORD=your_db_password
Step 3: Create Migrations for `hotels` and `foods` Tables
Run migration commands:
 bash
php artisan make:migration create_hotels_table
php artisan make:migration create_foods_table
Update Migrations:
Hotels Table:
 php
Schema::create('hotels', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('location');
    $table->timestamps();
});
Foods Table:
  php
Schema::create('foods', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->decimal('price', 8, 2);
    $table->foreignId('hotel_id')->constrained('hotels')->onDelete('cascade');
    $table->timestamps();
});
Run migrations:
 bash
php artisan migrate
Step 4: Models
 Hotel Model:
 php
class Hotel extends Model
{
    protected $fillable = ['name', 'location'];

    public function foods()
    {
        return $this->hasMany(Food::class);
    }
}
Food Model:
php
class Food extends Model
{
    protected $fillable = ['name', 'price', 'hotel_id'];

    public function hotel()
    {
        return $this->belongsTo(Hotel::class);
    }
}
Step 5: Controllers
Run commands to create controllers:
 bash
php artisan make:controller HotelController
php artisan make:controller FoodController
HotelController:
php
public function index(Request $request)
{
    $query = Hotel::query();

    if ($request->has('search')) {
        $query->where('name', 'like', '%' . $request->search . '%');
    }
    if ($request->has('sort')) {
        $query->orderBy('name', $request->sort);
    }

    return response()->json($query->get());
}
FoodController:
 php
public function index(Request $request)
{
    $query = Food::with('hotel');

    if ($request->has('search')) {
        $query->where('name', 'like', '%' . $request->search . '%');
    }
    if ($request->has('sort')) {
        $query->orderBy('price', $request->sort);
    }

    return response()->json($query->get());
}
Dashboard Logic (Add in HotelController):
php
public function dashboard()
{
    $hotelsCount = Hotel::count();
    $foodsCount = Food::count();
    $monthlyHotels = Hotel::selectRaw('MONTH(created_at) as month, COUNT(*) as count')
                          ->groupBy('month')->get();
    return response()->json([
        'hotelsCount' => $hotelsCount,
        'foodsCount' => $foodsCount,
        'monthlyHotels' => $monthlyHotels
    ]);
}
Step 6: Routes
Add routes in routes/api.php`:
php
Route::get('/hotels', [HotelController::class, 'index']);
Route::get('/foods', [FoodController::class, 'index']);
Route::get('/dashboard', [HotelController::class, 'dashboard']);
2. Frontend Setup (React)
Step 1: Setup React
Create a new React project:
  bash
npx create-react-app frontend
cd frontend
npm install axios react-chartjs-2 chart.js
Step 2: Create Pages
Hotel Listing Page (`HotelList.js`)
jsx
import React, { useEffect, useState } from 'react';
import axios from 'axios';
const HotelList = () => {
  const [hotels, setHotels] = useState([]);
  const [search, setSearch] = useState('');
  const [sort, setSort] = useState('');
  useEffect(() => {
    axios.get('/api/hotels', { params: { search, sort } })
         .then(response => setHotels(response.data));
  }, [search, sort]);
  return (
    <div>
      <h1>Hotel Listing</h1>
      <input 
        type="text" 
        placeholder="Search Hotels" 
        onChange={(e) => setSearch(e.target.value)} 
      />
      <button onClick={() => setSort('asc')}>Sort Asc</button>
      <button onClick={() => setSort('desc')}>Sort Desc</button>
      <ul>
        {hotels.map(hotel => (
          <li key={hotel.id}>{hotel.name} - {hotel.location}</li>
        ))}
      </ul>
    </div>
  );
};

export default HotelList;
Dashboard Page (`Dashboard.js`)
  jsx
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import { Bar } from 'react-chartjs-2';
const Dashboard = () => {
  const [data, setData] = useState({ hotelsCount: 0, foodsCount: 0, monthlyHotels: [] });
  useEffect(() => {
    axios.get('/api/dashboard').then(res => setData(res.data));
  }, []);
  const chartData = {
    labels: data.monthlyHotels.map(item => `Month ${item.month}`),
    datasets: [{ label: 'Hotels Created', data: data.monthlyHotels.map(item => item.count) }]
  };
  return (
    <div>
      <h1>Dashboard</h1>
      <p>Total Hotels: {data.hotelsCount}</p>
      <p>Total Foods: {data.foodsCount}</p>
      <Bar data={chartData} />
    </div>
  );
};
export default Dashboard;
Step 3: Connect Backend and Frontend
1. Start Laravel backend:
  bash
php artisan serve
2. Start React frontend:
bash
npm start
3. Zipping the Code
 Zip the Laravel backend (`hotel-food-app`) and React frontend (`frontend`) excluding `node_modules` and `vendor` folders.


 
