---
title: music-dl(2) 프로젝트 구성과정
date: 2021-08-21 19:28:00
tags:
---

이전 글에서는 music-dl 프로젝트를 시작하게 된 이유를 아주 길게 설명했다.
이번에는 프로젝트의 구성과 점진적으로 개선한 과정을 설명하는 글을 써보려고 한다.
이 글은 더 나은 음악감상을 위해 나처럼 헤매는 사람들이 시도해볼 수 있게 하는 목적으로 최대한 친절하게 써보려고 노력했다. 프로그래밍 경험이 있으신 분들이라면 그리 어렵지 않게 할 수 있을 정도로 간단한 프로젝트이긴 하다. 

내가 가장 처음에 했던 방법은 `youtube-dl --list-formats https://www.youtube.com/watch\?v=[VIDEOID]` 로 다운로드 가능한 포맷을 알아보는 거였다.

다음과 같은 결과물을 얻을 수 있다.
```
format code  extension  resolution note
249          webm       audio only tiny   59k , opus @ 50k (48000Hz), 1.22MiB
250          webm       audio only tiny   73k , opus @ 70k (48000Hz), 1.61MiB
140          m4a        audio only tiny  130k , m4a_dash container, mp4a.40.2@128k (44100Hz), 3.26MiB
251          webm       audio only tiny  137k , opus @160k (48000Hz), 3.17MiB
160          mp4        256x144    144p   56k , avc1.4d400c, 24fps, video only, 1.15MiB
394          mp4        256x144    144p   69k , av01.0.00M.08, 24fps, video only, 1.64MiB
278          webm       256x144    144p   96k , webm container, vp9, 24fps, video only, 2.15MiB
133          mp4        426x240    240p  121k , avc1.4d4015, 24fps, video only, 2.42MiB
242          webm       426x240    240p  129k , vp9, 24fps, video only, 2.66MiB
395          mp4        426x240    240p  138k , av01.0.00M.08, 24fps, video only, 3.06MiB
134          mp4        640x360    360p  219k , avc1.4d401e, 24fps, video only, 4.42MiB
243          webm       640x360    360p  224k , vp9, 24fps, video only, 4.62MiB
396          mp4        640x360    360p  272k , av01.0.01M.08, 24fps, video only, 6.04MiB
244          webm       854x480    480p  318k , vp9, 24fps, video only, 6.59MiB
135          mp4        854x480    480p  330k , avc1.4d401e, 24fps, video only, 6.73MiB
397          mp4        854x480    480p  459k , av01.0.04M.08, 24fps, video only, 10.21MiB
247          webm       1280x720   720p  523k , vp9, 24fps, video only, 10.94MiB
136          mp4        1280x720   720p  535k , avc1.4d401f, 24fps, video only, 10.46MiB
398          mp4        1280x720   720p  831k , av01.0.05M.08, 24fps, video only, 18.58MiB
18           mp4        640x360    360p  409k , avc1.42001E, 24fps, mp4a.40.2@ 96k (44100Hz), 10.28MiB (best)
```

오디오파일을 원하는 경우에는 format code가 251인 webm형식을 이용하면 된다.
`youtube-dl -f 251 https://www.youtube.com/watch\?v=[VIDEOID]` 를 실행하면 VIDEOID에 해당하는 영상을 지정한 포맷으로 다운로드할 수 있다.
하지만 이 과정 이후에 webm 형식의 파일을 mp3로 변환하는 추가적인 작업이 필요하다.
그리고 이 방법은 한개의 영상에 대해서만 가능하다. 플레이리스트에 있는 몇백개의 영상을 다운받기 위해서는 다른 방법이 필요하다.

youtube-dl은 플레이리스트를 지정해서 플레이리스트 전체를 다운로드할 수 있는 옵션을 지원한다.
그리고 특정 포맷을 지정하지 않고도 오디오파일로 다운로드하는 옵션과 webm을 mp3로 변환해주는 옵션도 있다.
플레이리스트별로 다운받으려면 각각의 디렉토리 지정이 필요하다. 파일명 포맷을 지정하는 옵션도 있어서 이러한 옵션들을 이용하면 다음과 같이 개선할 수 있다.

`youtube-dl --extract-audio --audio-format mp3 -o "playlist1/%(title)s.%(ext)s" https://www.youtube.com/playlist\?list\=[PLAYLISTID]`

플레이리스트 저장기능을 이대로 쓰면 노래를 한 곡만 추가해도 다음번엔 모든 노래를 다시 다운로드받는 문제가 있다.
youtube-dl에서 지원하는 --download-archive 옵션을 이용하면 새로 추가된 노래만 다운받을 수 있다.

`youtube-dl --extract-audio --audio-format mp3 --download-archive downloaded_file.txt -o "playlist1/%(title)s.%(ext)s" https://www.youtube.com/playlist\?list\=[PLAYLISTID]`
downloaded_file.txt 파일에는 다운받은 video_id들이 추가된다.

이 커맨드를 플레이리스트에 노래를 추가할 때마다 실행할 수 없으니, 일정시간마다 반복하기 위해서 cron job을 이용했다.
`crontab -e` 을 실행하고 `0 1 * * * /path/command.sh` 와 같은 형식으로 cronjob을 등록했다.
매일 새벽 1시에 `/path/command.sh`를 실행한다는 의미이다. cronjob의 시간 형식은 정말 다양하기 때문에 한번쯤 찾아보는걸 추천한다.
이제 매일매일 자동으로 내 플레이리스트에 추가된 노래를 지정된 디렉토리에 다운받을 수 있다. nPlayer같은 FTP를 지원하는 미디어플레이어 등을 이용해 서버와 연결해서 노래를 들을 수 있다. 개선의 여지가 많지만 일단은 한동안 이 방법에 큰 불편함 없이 잘 사용했다.