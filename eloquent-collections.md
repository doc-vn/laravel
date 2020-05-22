# Eloquent: Collections

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)
- [Tuỳ biến Collection](#custom-collections)

<a name="introduction"></a>
## Giới thiệu

Tất cả các kết quả được trả về bởi Eloquent đều là các instance của đối tượng `Illuminate\Database\Eloquent\Collection`, bao gồm cả các kết quả được truy xuất thông qua phương thức `get` hoặc được truy vấn thông qua quan hệ. Đối tượng collection Eloquent được extend từ [base collection](/docs/{{version}}/collections), do đó, nó thừa hưởng nhiều phương thức dùng để làm việc dễ dàng với mảng model Eloquent.

Tất nhiên, tất cả các collection cũng có vai trò như là một vòng lặp, cho phép bạn lặp qua nó như thể nó là một mảng PHP đơn thuần:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Tuy nhiên, collection mạnh mẽ hơn nhiều so với mảng và thêm nhiều phương thức map / reduce có thể được kết hợp với nhau bằng một giao diện dễ hiểu. Ví dụ: hãy xóa tất cả các model không hoạt động và lấy ra tên cho mỗi người dùng còn lại:

    $users = App\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} Trong khi hầu hết các phương thức Eloquent trả về một instance mới của collection Eloquent, thì các phương thức `pluck`, `keys`, `zip`, `collapse`, `flatten` và `flip` sẽ trả về một instance [base collection](/docs/{{version}}/collections). Tương tự, nếu mà phương thức `map` trả về một collection không chứa bất kỳ model Eloquent nào, nó sẽ tự động được chuyển về một base collection.

<a name="available-methods"></a>
## Các phương thức có sẵn

### The Base Collection

Tất cả các collection Eloquent đều được extend từ đối tượng [Laravel collection](/docs/{{version}}/collections); do đó, chúng kế thừa tất cả các phương thức mạnh mẽ được cung cấp bởi class collection:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](/docs/{{version}}/collections#method-all)
[average](/docs/{{version}}/collections#method-average)
[avg](/docs/{{version}}/collections#method-avg)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[combine](/docs/{{version}}/collections#method-combine)
[concat](/docs/{{version}}/collections#method-concat)
[contains](/docs/{{version}}/collections#method-contains)
[containsStrict](/docs/{{version}}/collections#method-containsstrict)
[count](/docs/{{version}}/collections#method-count)
[crossJoin](/docs/{{version}}/collections#method-crossjoin)
[dd](/docs/{{version}}/collections#method-dd)
[diff](/docs/{{version}}/collections#method-diff)
[diffKeys](/docs/{{version}}/collections#method-diffkeys)
[dump](/docs/{{version}}/collections#method-dump)
[each](/docs/{{version}}/collections#method-each)
[eachSpread](/docs/{{version}}/collections#method-eachspread)
[every](/docs/{{version}}/collections#method-every)
[except](/docs/{{version}}/collections#method-except)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatMap](/docs/{{version}}/collections#method-flatmap)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[isNotEmpty](/docs/{{version}}/collections#method-isnotempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[mapInto](/docs/{{version}}/collections#method-mapinto)
[mapSpread](/docs/{{version}}/collections#method-mapspread)
[mapToGroups](/docs/{{version}}/collections#method-maptogroups)
[mapWithKeys](/docs/{{version}}/collections#method-mapwithkeys)
[max](/docs/{{version}}/collections#method-max)
[median](/docs/{{version}}/collections#method-median)
[merge](/docs/{{version}}/collections#method-merge)
[min](/docs/{{version}}/collections#method-min)
[mode](/docs/{{version}}/collections#method-mode)
[nth](/docs/{{version}}/collections#method-nth)
[only](/docs/{{version}}/collections#method-only)
[pad](/docs/{{version}}/collections#method-pad)
[partition](/docs/{{version}}/collections#method-partition)
[pipe](/docs/{{version}}/collections#method-pipe)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[split](/docs/{{version}}/collections#method-split)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[tap](/docs/{{version}}/collections#method-tap)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[union](/docs/{{version}}/collections#method-union)
[unique](/docs/{{version}}/collections#method-unique)
[uniqueStrict](/docs/{{version}}/collections#method-uniquestrict)
[unless](/docs/{{version}}/collections#method-unless)
[values](/docs/{{version}}/collections#method-values)
[when](/docs/{{version}}/collections#method-when)
[where](/docs/{{version}}/collections#method-where)
[whereStrict](/docs/{{version}}/collections#method-wherestrict)
[whereIn](/docs/{{version}}/collections#method-wherein)
[whereInStrict](/docs/{{version}}/collections#method-whereinstrict)
[whereNotIn](/docs/{{version}}/collections#method-wherenotin)
[whereNotInStrict](/docs/{{version}}/collections#method-wherenotinstrict)
[zip](/docs/{{version}}/collections#method-zip)

</div>

<a name="custom-collections"></a>
## Tuỳ biến Collection

Nếu bạn cần sử dụng một đối tượng tùy biến `Collection` với các phương thức được mở rộng của riêng bạn, bạn có thể ghi đè phương thức `newCollection` trên model của bạn:

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

Khi bạn đã định nghĩa một phương thức `newCollection`, bạn sẽ nhận lại được một instance của collection tùy biến của bạn bất cứ khi nào Eloquent trả về một instance `Collection` cho model đó. Nếu bạn muốn sử dụng collection tùy biến này cho mọi model trong application của bạn, bạn nên ghi đè phương thức `newCollection` trên một class model cơ sở được extend cho tất cả các model của bạn.
