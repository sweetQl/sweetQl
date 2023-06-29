- 👋 Hi, I’m @sweetQl
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
sweetQl/sweetQl is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
获取广东省人民政府办公厅特定日期间的政策数据。
地址: https://www.gd.gov.cn/gkmlpt/policy
  
 import requests
from lxml import etree


def get_policy_data(start_date, end_date):
    url = "https://www.gd.gov.cn/gkmlpt/policy"

    # 构造查询参数
    params = {
        "pi": 1,  # 页码
        "ps": 20,  # 每页显示数量
        "sort": "uptime",  # 按更新时间排序
        "dir": "1",  # 升序排列
        "directorytreeid": "21725",  # 广东省人民政府办公厅目录ID
        "starttime": start_date,  # 开始日期
        "endtime": end_date  # 结束日期
    }

    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    }

    # 发送GET请求获取网页内容
    response = requests.get(url,
                            params=params,
                            headers=headers)

    if response.status_code == 200:
        # 使用lxml解析网页内容
        html = etree.HTML(response.text)

        # 获取政策列表
        policy_list = html.xpath("//div[@class='detail-more']/ul/li")

        for policy in policy_list:
            # 获取政策信息
            index = policy.xpath("./span[1]/text()")[0]  # 索引号
            institution = policy.xpath("./span[2]/text()")[0]  # 发布机构
            publish_date = policy.xpath("./span[3]/text()")[0]  # 发布日期
            title = policy.xpath("./a/@title")[0]  # 政策标题
            text_url = policy.xpath("./a/@href")[0]  # 政策正文文本链接

            # 获取政策正文文本
            text_response = requests.get(text_url,
                                         headers=headers)

            if text_response.status_code == 200:
                # 解析政策正文文本
                text_html = etree.HTML(text_response.text)
                text = text_html.xpath("//div[@id='printContent']//text()")
                text = " ".join(text).strip()
            else:
                text = ""

            # 获取政策正文附件链接
            attachment_url = policy.xpath("./span[4]/a/@href")
            if attachment_url:
                attachment_url = attachment_url[0]

            # 输出政策信息
            print("索引号:",
                  index)
            print("发布机构:",
                  institution)
            print("发布日期:",
                  publish_date)
            print("政策标题:",
                  title)
            print("政策正文文本:",
                  text)
            print("政策正文附件链接:",
                  attachment_url)
            print("=" * 50)

    else:
        print("请求失败:",
              response.status_code)


# 输入日期范围
start_date = input("请输入开始日期(格式：YYYYMMDD): ")
end_date = input("请输入结束日期(格式：YYYYMMDD): ")

# 获取政策数据
get_policy_data(start_date,
                end_date)



编写爬虫程序获取天眼查平台某家公司的全部专利数据。
地址https://www.tianyancha.com/



import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def login(username, password):
    # 创建浏览器实例
    driver = webdriver.Chrome()
    driver.maximize_window()

    # 打开天眼查平台
    driver.get('https://www.tianyancha.com/')

    # 等待登录按钮加载完成
    login_button = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.LINK_TEXT, '登录/注册'))
    )

    # 点击登录按钮
    login_button.click()

    # 输入用户名和密码并登录
    username_input = driver.find_element(By.NAME, 'autoLogin')
    password_input = driver.find_element(By.NAME, 'pwd')
    submit_button = driver.find_element(By.XPATH, '//button[@type="submit"]')

    username_input.send_keys(username)
    password_input.send_keys(password)
    submit_button.click()

    # 等待登录成功
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.XPATH, '//a[@data-id="nav_patent"]'))
    )

    # 获取Cookie
    cookies = driver.get_cookies()

    # 关闭浏览器
    driver.quit()

    return cookies

def get_patent_data(company_name, cookies):
    # 创建浏览器实例
    driver = webdriver.Chrome()
    driver.maximize_window()

    # 打开天眼查平台
    driver.get('https://www.tianyancha.com/')

    # 设置Cookie
    for cookie in cookies:
        driver.add_cookie(cookie)

    # 刷新页面
    driver.refresh()

    # 输入企业名称并搜索
    search_input = driver.find_element(By.XPATH, '//input[@placeholder="请输入公司名称、人名"]')
    search_input.send_keys(company_name)
    search_button = driver.find_element(By.XPATH, '//i[@class="layui-icon layui-icon-search"]')
    search_button.click()

    # 等待搜索结果加载完成
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.XPATH, '//a[@class="name ellipsis-1"]'))
    )

    # 点击进入公司页面
    company_link = driver.find_element(By.XPATH, '//a[@class="name ellipsis-1"]')
    company_link.click()

    # 等待专利数据加载完成
    time.sleep(5)  # 假设等待5秒钟，可以根据实际情况调节等待时间

    # 获取专利数据列表
    patent_list = driver.find_elements(By.XPATH, '//div[@class="index-card-item border-2"]')

    # 解析专利数据
    patent_data = []
    for patent in patent_list:
        patent_title = patent.find_element(By.XPATH, '//h3[@class="tyc-num lh24"]')
        patent_info = patent.find_elements(By.XPATH, '//div[@class="index-info-item"]')
        
        patent_data.append({
            'title': patent_title.text,
            'info': [info.text for info in patent_info]
        })

    # 关闭浏览器
    driver.quit()

    return patent_data

# 主程序
username = 'your_username'
password = 'your_password'
company_name = '华为技术有限公司'

# 登录并获取Cookie
cookies = login(username, password)

# 获取指定企业的专利数据
patent_data = get_patent_data(company_name, cookies)

# 输出结果
for patent in patent_data:
    print('专利标题:', patent['title'])
    print('专利信息:', patent['info'])
    print('--------------------------------------------')

                
