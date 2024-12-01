# Refactoring code that accesses external services

## 一些重构手法

Bounded Context

## 



```ruby
# 原始代码
class VideoService…

  def video_list
    @video_list = JSON.parse(File.read('videos.json'))
    ids = @video_list.map{|v| v['youtubeID']}
    client = GoogleAuthorizer.new(
      token_key: 'api-youtube',
      application_name: 'Gateway Youtube Example',
      application_version: '0.1'
      ).api_client
    youtube = client.discovered_api('youtube', 'v3')
    request = {
      api_method: youtube.videos.list,
      parameters: {
        id: ids.join(","),
        part: 'snippet, contentDetails, statistics',
      }
    }
    response = JSON.parse(client.execute!(request).body)
    ids.each do |id|
      video = @video_list.find{|v| id == v['youtubeID']}
      youtube_record = response['items'].find{|v| id == v['id']}
      video['views'] = youtube_record['statistics']['viewCount'].to_i
      days_available = Date.today - Date.parse(youtube_record['snippet']['publishedAt'])
      video['monthlyViews'] = video['views'] * 365.0 / days_available / 12      
    end
    return JSON.dump(@video_list)
  end
```



```ruby
class VideoService…

  def video_list
    @video_list = JSON.parse(File.read('videos.json'))
    ids = @video_list.map{|v| v['youtubeID']}
    response = call_youtobe ids
    ids.each do |id|
      video = @video_list.find{|v| id == v['youtubeID']}
      youtube_record = response['items'].find{|v| id == v['id']}
      video['views'] = youtube_record['statistics']['viewCount'].to_i
      days_available = Date.today - Date.parse(youtube_record['snippet']['publishedAt'])
      video['monthlyViews'] = video['views'] * 365.0 / days_available / 12      
    end
    return JSON.dump(@video_list)
  end

  # extract method
  #     
  def call_youtobe ids
    client = GoogleAuthorizer.new(
      token_key: 'api-youtube',
      application_name: 'Gateway Youtube Example',
      application_version: '0.1'
    ).api_client
    youtube = client.discovered_api('youtube', 'v3')
    request = {
      api_method: youtube.videos.list,
      parameters: {
        id: ids.join(","),
        part: 'snippet, contentDetails, statistics',
      }
    }
    response = JSON.parse(client.execute!(request).body)
```

