import requests
import json
from itertools import permutations


API_KEY = "7288dfc74ec3273f1f9a722e297557e7"
WS_URL = "https://api.openweathermap.org/data/2.5/forecast"


class City:
  "Represents a city"

  def __init__(self, name, temperatures):
      self.name = name
      self.temps = temperatures

  def get_temperature(self, day):
      return self.temps[day]

  def __str__(self):
      return self.name


class Route:
  "Represents a sequence of cities"
  
  def __init__(self, cities_in_route):
    self.cities = cities_in_route

  def avg_max_temps(self):
    avgtemp = 0
    for j in range(len(self.cities)):
      avgtemp += self.cities[j].get_temperature(j)
    avgtemp/=len(self.cities)
    return avgtemp

  def __str__(self):  
    return " : ".join(str(city) for city in self.cities) 


def fetch_weather(id):
  "Retrieves forecast for daily max temps of a city for next 5 days"

  query_string = "?id={}&units=imperial&APIKEY={}".format(id, API_KEY) #added to end of URL to request info for city id
  request_url = WS_URL + query_string
  
  try:
    response = requests.get(request_url)
    if response.status_code == 200:
      d = response.json()
      city_name = d["city"]['name']
      lst = d['list']
      tmp_list = []
      for i in range(len(lst) // 8):
          li = [x for x in range(len(lst)) if x // 8 == i]
          tmp_list.append(max([lst[j]["main"]["temp_max"] for j in li]))       
    return City(city_name, tmp_list)

  except requests.ConnectionError as e:
    print("Connection Error. Make sure you are connected to Internet. Technical Details given below.\n")
    print(str(e)) 
   
  except ValueError:
    print("Incorrect data found in .json file. Please check your file")

  except IOError:
    print ("Could not read the file")
  
  except:
    print ("Request failed")


if __name__ == "__main__":

  print("Here are all the possible routes and their average temperatures:\n")

  id_list = json.loads(open("cities.json").read()) #retrieve city id's from .json file
  possible_routes = list(permutations(id_list)) #creates all possible routes
  lowest_temp = 1000 #not a great way to do this...sets lowest temp to something unreasonably high for comparison. needs improvement
  best_route = None

  for a_route in possible_routes: #for a city list in the permutation list
    cities = [fetch_weather(id) for id in a_route] #gets daily max temps for each city in a city list
    avg_temp = Route(cities).avg_max_temps() #gets total avg of city list
    current_route = Route(cities) #sets current route
  
    if lowest_temp >= avg_temp: #sets lowest_temp to lowest of avg temps
      lowest_temp = avg_temp
      best_route = current_route #sets best_route to route with lowest of avg temps

    #prints city names with total avg temp for route
    for i in range(len(cities)):
      city = cities[i]
      print(city) 
    print("%.2f" % avg_temp, "F\n")
  print("Lowest Average Temperature: ", "%.2f"% lowest_temp, "F\nBest route: ", best_route )
    
   


      
 


  
   
   

