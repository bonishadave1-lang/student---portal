
<html ng-app="studentApp">
<head>
    <meta charset="UTF-8">
    <title>ATMIYA Student Portal | Admin & Auth</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular-route.js"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    
    <style>
        :root { --atmiya-primary: #6a11cb; --atmiya-secondary: #2575fc; --glass: rgba(255, 255, 255, 0.95); }
        body { background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%); min-height: 100vh; font-family: 'Segoe UI', sans-serif; }
        .navbar { background: linear-gradient(to right, #6a11cb, #2575fc) !important; border-bottom: 4px solid #f4af1b; }
        .main-card { background: var(--glass); border-radius: 20px; backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.3); box-shadow: 0 10px 30px rgba(0,0,0,0.1); padding: 25px; }
        .login-card { max-width: 400px; margin: 100px auto; padding: 40px; }
        .table-container { max-height: 600px; overflow-y: auto; }
        .sticky-thead th { position: sticky; top: 0; background: #6a11cb !important; color: white; z-index: 10; }
        .details-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.75); display: flex; align-items: center; justify-content: center; z-index: 2000; }
        .details-modal { background: white; padding: 35px; border-radius: 25px; width: 90%; max-width: 500px; border-top: 10px solid var(--atmiya-primary); }
    </style>
</head>
<body ng-controller="GlobalController">

    <nav class="navbar navbar-expand-lg navbar-dark sticky-top" ng-if="currentUser">
        <div class="container">
            <a class="navbar-brand fw-bold" href="#/!">ATMIYA PORTAL {{currentUser.role === 'faculty' ? 'ADMIN' : 'STUDENT'}}</a>
            <div class="navbar-nav ms-auto align-items-center">
                <span class="text-white me-3 small">Welcome, {{currentUser.name}}</span>
                <a class="nav-link btn btn-warning btn-sm text-dark fw-bold px-3" href="#!add" ng-if="currentUser.role === 'faculty'">New Admission +</a>
                <button class="btn btn-light btn-sm ms-2" ng-click="logout()">Logout</button>
            </div>
        </div>
    </nav>

    <div class="container mt-4">
        <div ng-view></div>
    </div>

    <div class="details-overlay" ng-if="selectedStudent">
        <div class="details-modal shadow-lg">
            <h4 class="fw-bold mb-4 text-center">{{isEditMode ? 'Edit Record' : 'Student Info'}}</h4>
            <div class="mb-3">
                <label class="form-label small">Name</label>
                <input type="text" class="form-control" ng-model="selectedStudent.name" ng-disabled="!isEditMode">
            </div>
            <div class="row mb-3">
                <div class="col-6">
                    <label class="form-label small">Marks %</label>
                    <input type="number" class="form-control" ng-model="selectedStudent.marks" ng-disabled="!isEditMode">
                </div>
                <div class="col-6">
                    <label class="form-label small">Attendance %</label>
                    <input type="number" class="form-control" ng-model="selectedStudent.attendance" ng-disabled="!isEditMode">
                </div>
            </div>
            <div class="mb-4">
                <label class="form-label small">Course</label>
                <input type="text" class="form-control" ng-model="selectedStudent.course" ng-disabled="!isEditMode">
            </div>
            <div class="d-flex gap-2">
                <button class="btn btn-success w-100" ng-if="isEditMode" ng-click="saveUpdate()">Save Changes</button>
                <button class="btn btn-primary w-100" ng-click="closeDetails()">Close</button>
            </div>
        </div>
    </div>

    <script>
        var app = angular.module("studentApp", ["ngRoute"]);

        app.config(function($routeProvider) {
            $routeProvider
            .when("/login", {
                template: `
                    <div class="main-card login-card text-center">
                        <h2 class="fw-bold mb-4">Portal Login</h2>
                        <form ng-submit="doLogin()">
                            <input type="text" class="form-control mb-3" placeholder="Username (student1, student2...)" ng-model="loginData.user">
                            <input type="password" class="form-control mb-3" placeholder="Password (pass1, pass2...)" ng-model="loginData.pass">
                            <button type="submit" class="btn btn-primary w-100 py-2 mb-3">Login as Student</button>
                        </form>
                        <div class="hr-text text-muted mb-3">OR</div>
                        <button class="btn btn-outline-dark w-100 py-2" ng-click="facultyBypass()">Enter as Faculty (Admin)</button>
                        <p class="mt-3 small text-danger" ng-if="error">{{error}}</p>
                    </div>`,
                controller: "LoginController"
            })
            .when("/", {
                template: `
                    <div class="main-card" ng-if="currentUser.role === 'faculty'">
                        <div class="d-flex justify-content-between mb-3 align-items-center">
                            <h4 class="m-0 fw-bold">Faculty Management Panel ({{students.length}} Students)</h4>
                            <input type="text" class="form-control w-25" placeholder="Search by name or course..." ng-model="searchKey">
                        </div>
                        <div class="table-container border rounded">
                            <table class="table table-hover align-middle text-center m-0">
                                <thead class="sticky-thead">
                                    <tr>
                                        <th>Roll</th>
                                        <th class="text-start">Name</th>
                                        <th>Course</th>
                                        <th>Username</th>
                                        <th>Status</th>
                                        <th>Actions</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr ng-repeat="s in students | filter:searchKey">
                                        <td class="fw-bold">#{{s.roll}}</td>
                                        <td class="text-start">{{s.name}}</td>
                                        <td><span class="badge bg-info text-dark">{{s.course}}</span></td>
                                        <td><code>{{s.username}}</code></td>
                                        <td><span class="badge {{s.marks >= 35 ? 'bg-success':'bg-danger'}}">{{s.marks}}%</span></td>
                                        <td>
                                            <button class="btn btn-sm btn-outline-primary" ng-click="viewDetails(s, false)">View</button>
                                            <button class="btn btn-sm btn-dark" ng-click="viewDetails(s, true)">Edit</button>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <div class="main-card p-5 text-center" ng-if="currentUser.role === 'student'">
                        <div class="display-1 mb-4">🎓</div>
                        <h1 class="fw-bold">{{currentUser.name}}</h1>
                        <p class="badge bg-primary fs-6">{{currentUser.course}} Engineering</p>
                        <div class="row mt-4">
                            <div class="col-md-6 mb-3">
                                <div class="p-4 bg-light rounded-3 shadow-sm">
                                    <h5>Current Grade</h5>
                                    <h2 ng-class="currentUser.marks >= 35 ? 'text-success':'text-danger'">{{currentUser.marks}}%</h2>
                                </div>
                            </div>
                            <div class="col-md-6 mb-3">
                                <div class="p-4 bg-light rounded-3 shadow-sm">
                                    <h5>Attendance</h5>
                                    <h2 class="text-primary">{{currentUser.attendance}}%</h2>
                                </div>
                            </div>
                        </div>
                    </div>`,
                controller: "DashboardController"
            })
            .when("/add", {
                template: `
                    <div class="main-card mx-auto p-5" style="max-width: 500px;">
                        <h4 class="text-center mb-4 fw-bold text-primary">New Admission</h4>
                        <form ng-submit="save()">
                            <input type="text" class="form-control mb-3" placeholder="Full Name" ng-model="temp.name" required>
                            <select class="form-select mb-3" ng-model="temp.course" required>
                                <option value="">Select Course</option>
                                <option>Computer</option><option>Mechanical</option><option>Civil</option><option>Electrical</option>
                            </select>
                            <input type="number" class="form-control mb-3" placeholder="Marks %" ng-model="temp.marks" required>
                            <input type="number" class="form-control mb-3" placeholder="Attendance %" ng-model="temp.attendance" required>
                            <button type="submit" class="btn btn-primary w-100 py-3 mt-2 shadow">REGISTER STUDENT</button>
                            <a href="#/!" class="btn btn-link w-100 mt-2">Cancel</a>
                        </form>
                    </div>`,
                controller: "AddController"
            })
            .otherwise({ redirectTo: "/login" });
        });

        app.service("DataStore", function() {
            var DB_KEY = "atmiya_v6_names";
            var AUTH_KEY = "atmiya_session";

            this.sync = function(d) { localStorage.setItem(DB_KEY, JSON.stringify(d)); };
            
            this.init = function() {
                var saved = localStorage.getItem(DB_KEY);
                if (saved) return JSON.parse(saved);

                var data = [];
                var courses = ["Computer", "Mechanical", "Civil", "Electrical"];
                var firstNames = ["Aarav", "Vihaan", "Aditya", "Sai", "Arjun", "Ishaan", "Aaryan", "Krishna", "Rahul", "Ananya", "Diya", "Priya", "Sana", "Riya", "Aavya"];
                var lastNames = ["Patel", "Sharma", "Joshi", "Verma", "Mehta", "Shah", "Guptha", "Reddy", "Nair", "Mishra"];

                for (var i = 1; i <= 400; i++) {
                    var fName = firstNames[Math.floor(Math.random() * firstNames.length)];
                    var lName = lastNames[Math.floor(Math.random() * lastNames.length)];
                    
                    data.push({
                        roll: i,
                        name: fName + " " + lName + " (" + i + ")",
                        course: courses[Math.floor((i-1) / 100)],
                        marks: Math.floor(Math.random() * 60) + 30,
                        attendance: Math.floor(Math.random() * 25) + 70,
                        username: "student" + i,
                        password: "pass" + i
                    });
                }
                this.sync(data);
                return data;
            };

            this.students = this.init();
            this.setSession = function(u) { sessionStorage.setItem(AUTH_KEY, JSON.stringify(u)); };
            this.getSession = function() { return JSON.parse(sessionStorage.getItem(AUTH_KEY)); };
            this.clearSession = function() { sessionStorage.removeItem(AUTH_KEY); };
        });

        app.controller("GlobalController", function($scope, $location, DataStore, $rootScope) {
            $scope.students = DataStore.students;
            $rootScope.currentUser = DataStore.getSession();

            $scope.logout = function() {
                DataStore.clearSession();
                $rootScope.currentUser = null;
                $location.path("/login");
            };

            $scope.viewDetails = function(s, editMode) { 
                $scope.selectedStudent = angular.copy(s);
                $scope.originalStudent = s;
                $scope.isEditMode = editMode; 
            };

            $scope.saveUpdate = function() {
                var index = $scope.students.indexOf($scope.originalStudent);
                $scope.students[index] = $scope.selectedStudent;
                DataStore.sync($scope.students);
                $scope.selectedStudent = null;
            };

            $scope.closeDetails = function() { $scope.selectemdStudent = null; };
        });

        app.controller("LoginController", function($scope, $location, DataStore, $rootScope) {
            if($rootScope.currentUser) $location.path("/");
            $scope.doLogin = function() {
                var user = $scope.students.find(s => s.username === $scope.loginData.user && s.password === $scope.loginData.pass);
                if (user) {
                    $rootScope.currentUser = { ...user, role: 'student' };
                    DataStore.setSession($rootScope.currentUser);
                    $location.path("/");
                } else { $scope.error = "Invalid Credentials! Use student1 / pass1"; }
            };
            $scope.facultyBypass = function() {
                $rootScope.currentUser = { name: "Admin Faculty", role: "faculty" };
                DataStore.setSession($rootScope.currentUser);
                $location.path("/");
            };
        });

        app.controller("DashboardController", function($scope, $rootScope, $location) {
            if (!$rootScope.currentUser) $location.path("/login");
        });

        app.controller("AddController", function($scope, $location, DataStore, $rootScope) {
            if (!$rootScope.currentUser || $rootScope.currentUser.role !== 'faculty') $location.path("/");
            $scope.save = function() {
                var nextID = $scope.students.length + 1;
                $scope.temp.roll = nextID;
                $scope.temp.username = "student" + nextID;
                $scope.temp.password = "pass" + nextID;
                $scope.students.push($scope.temp);
                DataStore.sync($scope.students);
                $location.path("/");
            };
        });
    </script>
</body>
</html>
