# 게시물 리스트 조회 쿼리
## 검색 키워드 timestampdiff, case
       SELECT p.postIdx as postIdx,
              u.userIdx as userIdx,
              u.nickName as nickName,
              u.profileImgUrl as profileImgUrl,
              p.content as content,
              IF(postLikeCount is null, 0, postLikeCount) as postLikeCount,
              IF(commentCount is null, 0, commentCount) as commentCount,
              case
                   when timestampdiff(second, p.updatedAt, current_timestamp) < 60
                       then concat(timestampdiff(second, p.updatedAt, current_timestamp), '초 전')
                   when timestampdiff(minute , p.updatedAt, current_timestamp) < 60
                       then concat(timestampdiff(minute, p.updatedAt, current_timestamp), '분 전')
                   when timestampdiff(hour , p.updatedAt, current_timestamp) < 24
                       then concat(timestampdiff(hour, p.updatedAt, current_timestamp), '시간 전')
                   when timestampdiff(day , p.updatedAt, current_timestamp) < 365
                       then concat(timestampdiff(day, p.updatedAt, current_timestamp), '일 전')
                   else timestampdiff(year , p.updatedAt, current_timestamp)
               end as uploadTime,
              IF(pl.status = 'ACTIVE', 'Y', 'N') as likeOrNot
       FROM Post as p
           join User as u on u.userIdx = p.userIdx
           left join (select postIdx, count(postLikeidx) as postLikeCount from PostLike WHERE status = 'ACTIVE' group by postIdx) plc on plc.postIdx = p.postIdx
           left join (select postIdx, count(commentIdx) as commentCount from Comment WHERE status = 'ACTIVE' group by postIdx) c on c.postIdx = p.postIdx
           left join Follow as f on f.followeeIdx = p.userIdx and f.status = 'ACTIVE'
           left join PostLike as pl on pl.userIdx = f.followerIdx and pl.postIdx = p.postIdx
       WHERE f.followerIdx = ? and p.status = 'ACTIVE'
       group by p.postIdx
