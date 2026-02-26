from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import random
import pandas as pd 

driver = webdriver.Chrome()
seen = set()
page = 1
information = []

while True:    
    url = f"https://clutch.co/us/developers/ecommerce?page={page}"

    driver.get(url)

    time.sleep(random.uniform(3.4,5.4))

    data = WebDriverWait(driver,19).until((
        EC.presence_of_all_elements_located((By.CLASS_NAME,"provider-list-item"))
    ))
       
    
    if not data:
        print("No companies found. Stopping scraper.")
        break

    for d in data[3:]:
        try:
            name = (d.find_element(By.CSS_SELECTOR,"a.provider__title-link")).text
           
            
        except:
            name = "N|A"


            if not name or name  in seen:
                continue
            seen.add(name)
        is_featured = d.get_attribute("data-featured")
        if is_featured == "true" and page > 1:
            continue
            
        try:

            p_url=(d.find_element(By.CSS_SELECTOR,"a.provider__title-link")).get_attribute("href")
        except:
            p_url = "not there"
        try:

            rating = (d.find_element(By.CSS_SELECTOR,".provider__rating.sg-rating.rating-reviews")).text.strip()
        except:
            rating = "not there"

        try:

            min_project_size = (d.find_element(By.CSS_SELECTOR,".provider__highlights-item.sg-tooltip-v2.min-project-size")).text.strip()
        except:
            min_project_size = "not there"
        try:

            employee_count = (d.find_element(By.CSS_SELECTOR,".provider__highlights-item.sg-tooltip-v2.employees-count")).text.strip()
        except:
            employee_count = "not there"
        try:

            hourly_rate = (d.find_element(By.CSS_SELECTOR,".provider__highlights-item.sg-tooltip-v2.hourly-rate")).text.strip()
        except:
            hourly_rate = "not there"
        try:

            location = (d.find_element(By.CSS_SELECTOR,".provider__highlights-item.sg-tooltip-v2.location")).text.strip()
        except:
            location = "not there"
        try:

            services = (d.find_element(By.CSS_SELECTOR,".provider__services-list")).text.strip()
        except:
            services = "not there"
        if name not in seen:
            seen.add(name)
            print(name)
        
            information.append({
                "Name" : name,
                "p_url" : p_url,
                "rating" : rating,
                "min_project_size" : min_project_size,
                "employee_count" : employee_count,
                "hourly_rate" : hourly_rate,
                "location" : location,
                "services" : services,
                })
    print(f"Page {page} scraped. Total unique companies: {len(information)}")    
    page += 1

    if page == 100:
        break

df = pd.DataFrame(information)
print(df)
df.to_csv("main.csv",index=False)
   
driver.close()
