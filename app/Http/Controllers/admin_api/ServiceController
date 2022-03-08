<?php

namespace App\Http\Controllers\admin_api;

use App\Http\Controllers\Controller;
use App\Models\Block_steps;
use App\Models\Npick_block_withdraws;
use App\Models\Settings;
use App\Models\Users;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\DB;

class ServiceController extends Controller
{
    /**
     * N+ 포인트 출금 신청내역 조회
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function block_withdraw(Request $request)
    {
        $user = Auth()->user();

        $date = $request->input('date');
        $status = $request->input('status');

        $where = [];
        if ($date) {
            $where[] = ['npick_block_withdraws.created_at', '>=', $date. ' 00:00:00'];
            $where[] = ['npick_block_withdraws.created_at', '<=', $date. ' 23:59:59'];
        }
        if (isset($status)) {
            $where['npick_block_withdraws.status'] = $status;

        }

        $withdraws = Npick_block_withdraws::selectRaw('npick_block_withdraws.id, if(users.email is not null,users.email,concat(users.provider,"@",users.provider_id)) as email, users.name, users.nickname, users.phone, npick_block_withdraws.npick_block, npick_block_withdraws.wallet,npick_block_withdraws.status, npick_block_withdraws.created_at')
            ->where($where)
            ->leftJoin('users','users.id','=','npick_block_withdraws.user_id')
            ->orderBy('npick_block_withdraws.id', 'DESC')
            ->paginate(10);

        $data = [];
        $data['withdraws'] = $withdraws;

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }

    /**
     * N+ 포인트 출석 완료 스텝별 지급 개수 커스텀 설정 조회
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function block_step(Request $request)
    {
        $step = $request->input('step');

        if ($step) {
            $step_info = Block_steps::select('id', 'step', 'block')
                ->where('step',$step)
                ->orderBy('id','DESC')
                ->first();

            $data = [];
            $data['step'] = $step_info;
        } else {
            $step_lists = Block_steps::select('id', 'step', 'block')
                ->get();

            $data = [];
            $data['step_lists'] = $step_lists;
        }
        
        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }

    /**
     * N+ 포인트 출석 완료 스텝별 지급 개수 커스텀 설정 변경
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function step_modify(Request $request){
        $user = Auth()->user();

        $steps = $request->input('step');
        $block = $request->input('block');

        Block_steps::where('step','!=',null)->delete();
        foreach ($steps as $key => $step) {
            if (isset($block[$key])) {
                $bs = new Block_steps();
                $bs->step = $step;
                $bs->block = $block[$key];
                $bs->created_by = $user->id;
                $bs->updated_by = $user->id;
                $bs->save();
            }
        }

        $data = [];

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }

    /**
     * N+ 포인트 기본 출석 / 홀수 스텝 출석 완료 / 짝수 스텝 출석 완료 지급 개수 조회
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function block_count(Request $request)
    {
        $default = Settings::where('option_name', 'default_block')->select('option_value')->first();
        $odd = Settings::where('option_name', 'odd_step')->select('option_value')->first();
        $even = Settings::where('option_name', 'even_step')->select('option_value')->first();

        $data = [];
        $data['default_block'] = $default->option_value;
        $data['odd_step'] = $odd->option_value;
        $data['even_step'] = $even->option_value;

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }
    
    /**
     * N+ 포인트 기본 출석 / 홀수 스텝 출석 완료 / 짝수 스텝 출석 완료 지급 개수 수정
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function block_modify(Request $request)
    {
        $user = Auth()->user();
        $default_block = $request->input('default');
        $odd = $request->input('odd');
        $even = $request->input('even');

        Settings::where('option_name', 'default_block')->update(['option_value' => $default_block]);
        Settings::where('option_name', 'odd_step')->update(['option_value' => $odd]);
        Settings::where('option_name', 'even_step')->update(['option_value' => $even]);

        $data = [];

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }
    
    /**
     * N+ 포인트 출금 신청 완료처리
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function block_withdraw_accept(Request $request)
    {
        $user = Auth()->user();

        $withdraw_id = $request->input('withdraw_id');

        if (!$withdraw_id) {
            return response()->json(['success' => false, 'alert' => '일시적인 오류로 실패하였습니다. 다시 시도해주세요.', 'data' => ''], 200);
        }

        $update = [];
        $update['status'] = 1;

        Npick_block_withdraws::where('id', $withdraw_id)->update($update);

        $data = [];

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }
}
